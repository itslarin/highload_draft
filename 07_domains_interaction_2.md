```mermaid
graph TD
    Identity[Identity & Access] -->|user_id JWT| Betting
    Identity -->|user_id JWT| UserProfile[User Profile]
    
    Market -->|MarketResolved| Resolution
    Market <-->|Sync: статус рынка| Betting
    
    Betting -->|BetPlaced| Wallet[Wallet & Ledger]
    Betting <-->|Sync: баланс| Wallet
    
    Resolution -->|PayoutProcessed| Wallet
    Resolution -->|обновление статистики| UserProfile
    
    Wallet -->|BalanceChanged| UserProfile
    
    Monetization -->|AdFreeActivated| Wallet
    Monetization -.->|обновление флага| UserProfile
    
    style Identity fill:#e1f5ff
    style Market fill:#fff4e1
    style Betting fill:#ffe1e1
    style Wallet fill:#e1ffe1
    style Resolution fill:#f0e1ff
    style UserProfile fill:#ffe1f0
    style Monetization fill:#ffffe1
