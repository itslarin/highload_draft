# Карта взаимодействия доменов (Context Map)

## Mermaid-диаграмма с типами вызовов

```mermaid
graph TD
    %% Легенда типов линий
    subgraph Legend["Легенда"]
        direction LR
        L1[Синхронный вызов] -->|REST/gRPC| L2[Немедленный ответ]
        L3[Асинхронное событие] -.->|Kafka/Queue| L4[Фоновая обработка]
    end
    
    %% Основные домены
    Identity[Identity & Access]
    Market[Market]
    Betting[Betting]
    Wallet[Wallet & Ledger]
    Resolution[Resolution]
    UserProfile[User Profile]
    Monetization[Monetization]
    
    %% Синхронные вызовы (сплошные стрелки)
    Identity -->|JWT валидация| Betting
    Identity -->|создание профиля| UserProfile
    Betting -->|проверка баланса| Wallet
    Betting -->|статус рынка| Market
    
    %% Асинхронные события (пунктирные стрелки)
    Market -.->|MarketResolved| Resolution
    Betting -.->|BetPlaced| Wallet
    Resolution -.->|PayoutProcessed| Wallet
    Wallet -.->|BalanceChanged| UserProfile
    Monetization -.->|AdFreeActivated| Wallet
    Monetization -.->|обновление флага| UserProfile
    
    %% Стили
    style Identity fill:#e1f5ff
    style Market fill:#fff4e1
    style Betting fill:#ffe1e1
    style Wallet fill:#e1ffe1
    style Resolution fill:#f0e1ff
    style UserProfile fill:#ffe1f0
    style Monetization fill:#ffffe1
    style Legend fill:#f5f5f5,stroke:#ccc
```


## Типы линий в диаграмме

### Синхронные вызовы (сплошная стрелка `-->`)
Используются для операций, требующих немедленного ответа.

* **Identity → Betting:** Валидация JWT при каждом запросе
* **Identity → UserProfile:** Создание профиля при регистрации
* **Betting → Wallet:** Проверка баланса перед размещением ставки
* **Betting → Market:** Валидация статуса рынка (OPEN/CLOSED)

**Технологии:** REST API, gRPC

### Асинхронные события (пунктирная стрелка `-.->`)
Используются для ослабления связности и масштабирования.

* **Market → Resolution:** Событие `MarketResolved` (рынок закрыт, известен результат)
* **Betting → Wallet:** Событие `BetPlaced` (ставка размещена, нужно списать средства)
* **Resolution → Wallet:** Событие `PayoutProcessed` (выплаты рассчитаны, нужно начислить)
* **Wallet → UserProfile:** Событие `BalanceChanged` (баланс изменился, обновить статистику)
* **Monetization → Wallet:** Событие `AdFreeActivated` (куплена подписка, нужно сжечь валюту)
* **Monetization → UserProfile:** Обновление флага `ad_free`

**Технологии:** Apache Kafka, RabbitMQ, Redis Pub/Sub
