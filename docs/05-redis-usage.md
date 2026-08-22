# 05 - Redis Kullanım Stratejisi

Redis en az 3 farklı amaçla kullanılacaktır:

### 1. Rate Limiting
- Tool başına event fırlatma limiti (token bucket / sliding window)
- IP başına Admin API limiti
- Key örneği: `ratelimit:tool:{tool_id}`, `ratelimit:ip:{ip}`
- TTL: pencere süresi kadar (örn. 60 saniye)

### 2. Routing Rule + Subscription Cache
- Her event geldiğinde DB’ye gitmemek için aktif rule ve subscription’lar cache’lenir.
- Key: `cache:subscriptions:{event_type_id}`
- TTL: 5 dakika
- Invalidation: Rule veya subscription değiştiğinde Kafka üzerinden “cache.invalidate” event’i yayınlanır + TTL yedek olarak kalır.

### 3. Distributed Lock
- Delivery Worker’da aynı delivery task’ın birden fazla instance tarafından işlenmesini engellemek için.
- Key: `lock:delivery:{delivery_attempt_id}`
- TTL: 30 saniye (işlem bitince release)

### Ek Kullanımlar (opsiyonel)
- Circuit breaker state (subscriber bazlı)
- Kısa süreli idempotency kontrolü

### TTL Stratejisi
Tüm key’lerde mutlaka TTL olacak. Memory şişmesi engellenecek.

### SPOF Riski ve Azaltma
Redis tek instance olduğunda SPOF’tur. 
- Geliştirme ortamında tek instance kabul edilebilir.
- Production’da Redis Sentinel veya Redis Cluster önerilir.
- Kritik path’lerde (rate limit) Redis down olursa “fail-open” veya “fail-closed” kararı config’den yönetilmeli.
