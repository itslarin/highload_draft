# Поток данных: Выбор рынка и размещение ставки

## Sequence Diagram

```mermaid
sequenceDiagram
    participant User as PWA (Frontend)
    participant GW as API Gateway
    participant MS as Market Service
    participant BE as Betting Engine
    participant WS as Wallet Service
    participant Redis as Redis Cache
    participant PG as PostgreSQL
    participant Kafka as Kafka
    participant WSS as WebSocket Server
    
    Note over User,WSS: === ФАЗА 1: Просмотр списка рынков ===
    
    User->>GW: GET /api/markets
    GW->>MS: forward request
    MS->>Redis: GET markets_list
    alt Cache HIT
        Redis-->>MS: cached data
    else Cache MISS
        MS->>PG: SELECT * FROM markets WHERE status='OPEN'
        PG-->>MS: markets data
        MS->>Redis: SET markets_list (TTL=60s)
    end
    MS-->>GW: markets list
    GW-->>User: 200 OK + markets
    
    Note over User,WSS: === ФАЗА 2: Выбор конкретного рынка ===
    
    User->>GW: GET /api/markets/{market_id}
    GW->>MS: forward request
    MS->>Redis: GET market:{market_id}
    alt Cache HIT
        Redis-->>MS: market details + odds
    else Cache MISS
        MS->>PG: SELECT m.*, o.*, mp.* FROM markets m JOIN outcomes o JOIN market_pools mp
        PG-->>MS: market data
        MS->>Redis: SET market:{market_id} (TTL=30s)
    end
    MS-->>GW: market details
    GW-->>User: 200 OK + market + odds
    
    Note over User,WSS: === ФАЗА 3: Подписка на real-time обновления ===
    
    User->>WSS: WebSocket connect (subscribe to market:{market_id})
    WSS->>Redis: SUBSCRIBE market:{market_id}:updates
    
    Note over User,WSS: === ФАЗА 4: Размещение ставки (CRITICAL PATH) ===
    
    User->>GW: POST /api/bets {market_id, outcome_id, amount: 100}
    GW->>GW: validate JWT token
    GW->>BE: forward bet request
    
    Note over BE: Валидация и проверка
    
    BE->>Redis: GET market:{market_id}:status
    Redis-->>BE: status=OPEN
    BE->>Redis: GET user:{user_id}:balance
    Redis-->>BE: balance=1000
    
    alt Balance insufficient
        BE-->>GW: 400 Bad Request (insufficient funds)
        GW-->>User: 400 Error
    else Market closed
        BE-->>GW: 400 Bad Request (market closed)
        GW-->>User: 400 Error
    else Validation passed
        BE->>PG: BEGIN TRANSACTION
        BE->>PG: UPDATE market_pools SET total_pool = total_pool + 100 WHERE market_id=? AND outcome_id=?
        PG-->>BE: pool updated
        BE->>PG: INSERT INTO bets (market_id, outcome_id, user_id, amount, odds_at_placement)
        PG-->>BE: bet created
        BE->>PG: UPDATE wallets SET balance = balance - 100 WHERE user_id=?
        PG-->>BE: balance updated
        BE->>PG: COMMIT
        
        Note over BE: Пересчет odds (Pari-Mutuel)
        
        BE->>PG: SELECT total_pool FROM market_pools WHERE market_id=?
        PG-->>BE: pools for all outcomes
        BE->>BE: calculate new odds for each outcome
        BE->>Redis: SET market:{market_id}:odds (new odds)
        BE->>Redis: SET user:{user_id}:balance (new balance)
        
        Note over BE: Публикация событий
        
        BE->>Kafka: publish BetPlaced {user_id, market_id, outcome_id, amount}
        BE->>Redis: PUBLISH market:{market_id}:updates {new_odds}
        
        BE-->>GW: 200 OK {bet_id, new_balance, new_odds}
        GW-->>User: 200 OK + bet confirmation
    end
    
    Note over User,WSS: === ФАЗА 5: Real-time обновление для других пользователей ===
    
    Redis->>WSS: message: new odds for market:{market_id}
    WSS->>User: WebSocket message: {new_odds}
    
    Note over User,WSS: === ФАЗА 6: Асинхронная обработка (фоновые задачи) ===
    
    Kafka->>WS: consume BetPlaced event
    WS->>PG: INSERT INTO ledger_transactions (user_id, type=BET, amount=-100)
    WS->>PG: UPDATE user_statistics SET total_bets = total_bets + 1
    
    Kafka->>MS: consume BetPlaced event
    MS->>Redis: UPDATE market:{market_id}:stats (volume, bet_count)
