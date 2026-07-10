```mermaid
graph TB
    subgraph Frontend["Frontend"]
        PWA[PWA Web/Mobile]
    end
    
    subgraph Gateway["Gateway Layer"]
        GW[API Gateway]
        WS[WebSocket Server]
    end
    
    subgraph Backend["Backend Services"]
        Auth[Identity]
        Profile[Profile]
        Market[Market]
        Betting[Betting]
        Wallet[Wallet]
        Resolution[Resolution]
        Monetization[Monetization]
    end
    
    subgraph Data["Data & Messaging"]
        PG[(PostgreSQL)]
        Redis[(Redis)]
        Kafka{{Kafka}}
    end
    
    PWA --> GW
    PWA --> WS
    GW --> Auth
    GW --> Market
    GW --> Betting
    GW --> Wallet
    GW --> Monetization
    WS --> Redis
    
    Betting --> Wallet
    Betting --> Market
    Market --> Kafka
    Betting --> Kafka
    Kafka --> Wallet
    Kafka --> Profile
    Kafka --> Resolution
    
    Auth --> PG
    Market --> PG
    Betting --> PG
    Wallet --> PG
    Market --> Redis
    Wallet --> Redis
