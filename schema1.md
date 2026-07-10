# Архитектурная диаграмма системы (System Design)

## Полная схема компонентов

```mermaid
graph TB
    %% Клиенты
    subgraph Clients["Клиенты"]
        Web[Web Browser]
        PWA[PWA Mobile]
    end
    
    %% Edge Layer
    subgraph Edge["Edge Layer"]
        CDN[CDN + WAF<br/>Cloudflare]
        GW[API Gateway<br/>Spring Cloud Gateway]
        WS[WebSocket Server<br/>Real-time Updates]
    end
    
    %% Микросервисы
    subgraph Services["Микросервисы (Spring Boot)"]
        Auth[Identity & Access<br/>Service]
        Profile[User Profile<br/>Service]
        Market[Market<br/>Service]
        Betting[Betting<br/>Engine]
        Wallet[Wallet & Ledger<br/>Service]
        Resolution[Resolution<br/>Service]
        Monetization[Monetization<br/>Service]
    end
    
    %% Data Layer
    subgraph Data["Data Layer"]
        PG[(PostgreSQL<br/>Sharded by market_id)]
        Redis[(Redis<br/>Cache + Pub/Sub)]
        Kafka{{Apache Kafka<br/>Event Bus}}
    end
    
    %% External
    subgraph External["External Services"]
        Scraper[Scraper Worker<br/>Polymarket API]
        OAuth[OAuth Providers<br/>Google, Apple]
        Ads[Ad Networks]
    end
    
    %% Client connections
    Web --> CDN
    PWA --> CDN
    CDN --> GW
    CDN --> WS
    
    %% Gateway routing
    GW --> Auth
    GW --> Profile
    GW --> Market
    GW --> Betting
    GW --> Wallet
    GW --> Monetization
    
    %% WebSocket
    WS -.->|Pub/Sub| Redis
    
    %% Sync calls (solid arrows)
    Auth -->|JWT validation| Profile
    Betting -->|check balance| Wallet
    Betting -->|check status| Market
    
    %% Async events (dashed arrows)
    Market -.->|MarketResolved| Kafka
    Betting -.->|BetPlaced| Kafka
    Resolution -.->|PayoutProcessed| Kafka
    Wallet -.->|BalanceChanged| Kafka
    Monetization -.->|AdFreeActivated| Kafka
    
    %% Kafka consumers
    Kafka -.-> Wallet
    Kafka -.-> Profile
    Kafka -.-> Resolution
    Kafka -.-> Market
    
    %% Database connections
    Auth --> PG
    Profile --> PG
    Market --> PG
    Betting --> PG
    Wallet --> PG
    Resolution --> PG
    Monetization --> PG
    
    %% Redis connections
    Market -->|cache odds| Redis
    Betting -->|cache pools| Redis
    Wallet -->|cache balances| Redis
    
    %% External integrations
    Scraper -->|fetch markets| Market
    Auth -->|OAuth| OAuth
    Monetization -->|serve ads| Ads
    
    %% Styles
    style Clients fill:#e1f5ff
    style Edge fill:#fff4e1
    style Services fill:#ffe1e1
    style Data fill:#e1ffe1
    style External fill:#f0e1ff
