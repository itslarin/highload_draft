Потоки данных (Data Flow)


Поток 1: Размещение ставки
User → Betting → [Sync] Wallet (проверка баланса)
              → [Async] Kafka: BetPlaced
              → Wallet (списание средств)
              → Market (обновление пула и odds)


Поток 2: Разрешение рынка
Scraper → Market: MarketResolved
        → [Async] Kafka: MarketResolved
        → Resolution (расчет выигрышей)
        → [Async] Kafka: PayoutProcessed
        → Wallet (начисление выигрышей)
        → User Profile (обновление статистики)


Поток 3: Покупка Ad-Free
User → Monetization: покупка подписки
     → [Async] Kafka: AdFreeActivated
     → Wallet (сжигание валюты)
     → User Profile (обновление флага ad_free)
