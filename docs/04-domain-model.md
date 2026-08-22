# 04 - Domain Model

## Entity Listesi ve Sorumlulukları

### 1. Tool
Platforma kayıtlı her bağımsız uygulama (publisher, subscriber veya her ikisi).

| Alan                    | Tip              | Açıklama |
|-------------------------|------------------|----------|
| id                      | UUID (PK)        | - |
| name                    | VARCHAR(100)     | Benzersiz tool adı |
| description             | TEXT             | Opsiyonel açıklama |
| base_url                | VARCHAR(500)     | Subscriber ise webhook base URL |
| status                  | ENUM             | active, inactive, suspended |
| api_key_hash            | VARCHAR(255)     | API Key’in hash’i (plaintext saklanmaz) |
| hmac_secret_hash        | VARCHAR(255)     | HMAC imza için kullanılan secret’ın hash’i |
| api_key_prefix          | VARCHAR(20)      | Key’in ilk karakterleri (gösterim için) |
| created_at              | TIMESTAMPTZ      | - |
| updated_at              | TIMESTAMPTZ      | - |
| created_by              | UUID             | Admin user id |

**Not:** ApiKey ve HmacSecret ayrı entity yapmak yerine Tool içinde tutuldu. Rotate işleminde yeni key üretilir, eski key grace period sonrası invalid edilir.

### 2. EventType
Bir tool’un yayınlayabileceği event tipleri.

| Alan                    | Tip              | Açıklama |
|-------------------------|------------------|----------|
| id                      | UUID (PK)        | - |
| tool_id                 | UUID (FK)        | Hangi tool yayınlıyor |
| name                    | VARCHAR(150)     | Örn: `document.created` |
| version                 | VARCHAR(20)      | `v1`, `v2` |
| schema                  | JSONB            | JSON Schema tanımı |
| description             | TEXT             | - |
| is_active               | BOOLEAN          | - |
| created_at              | TIMESTAMPTZ      | - |

### 3. Subscription
Bir tool’un hangi EventType’lara abone olduğu.

| Alan                    | Tip              | Açıklama |
|-------------------------|------------------|----------|
| id                      | UUID (PK)        | - |
| subscriber_tool_id      | UUID (FK)        | Abone olan tool |
| event_type_id           | UUID (FK)        | Abone olunan event tipi |
| webhook_path            | VARCHAR(300)     | Base URL + path (örn: `/webhooks/document`) |
| is_active               | BOOLEAN          | - |
| created_at              | TIMESTAMPTZ      | - |

Unique constraint: `(subscriber_tool_id, event_type_id)`

### 4. RoutingRule
Admin tarafından tanımlanan ek yönlendirme / filtreleme kuralları.

| Alan                    | Tip              | Açıklama |
|-------------------------|------------------|----------|
| id                      | UUID (PK)        | - |
| name                    | VARCHAR(150)     | - |
| event_type_id           | UUID (FK)        | Hangi event tipi için |
| subscriber_tool_id      | UUID (FK)        | Hedef subscriber |
| filter_expression       | JSONB            | Basit JSONPath / condition (örn: `{"status": "approved"}`) |
| priority                | INTEGER          | Yüksek öncelik önce uygulanır |
| is_active               | BOOLEAN          | - |
| created_at              | TIMESTAMPTZ      | - |

### 5. EventInstance
Platforma gelen her bir event kaydı (durable inbox).

| Alan                    | Tip              | Açıklama |
|-------------------------|------------------|----------|
| id                      | UUID (PK)        | Platformun ürettiği id |
| event_id                | UUID             | Client’ın gönderdiği idempotency key |
| event_type_id           | UUID (FK)        | - |
| source_tool_id          | UUID (FK)        | Publisher tool |
| payload                 | JSONB            | Asıl event içeriği |
| trace_id                | VARCHAR(100)     | Distributed tracing için |
| status                  | ENUM             | received, routed, completed, failed |
| received_at             | TIMESTAMPTZ      | - |
| raw_headers             | JSONB            | Debug için |

Unique constraint: `(source_tool_id, event_id)` → Idempotency

### 6. DeliveryAttempt
Bir event’in bir subscriber’a teslim edilme denemesi.

| Alan                    | Tip              | Açıklama |
|-------------------------|------------------|----------|
| id                      | UUID (PK)        | - |
| event_instance_id       | UUID (FK)        | - |
| subscription_id         | UUID (FK)        | - |
| attempt_number          | INTEGER          | 1, 2, 3... |
| status                  | ENUM             | pending, success, failed, retrying |
| http_status             | INTEGER          | - |
| response_body           | TEXT             | Kısa özet |
| latency_ms              | INTEGER          | - |
| error_message           | TEXT             | - |
| next_retry_at           | TIMESTAMPTZ      | - |
| created_at              | TIMESTAMPTZ      | - |

### 7. AuditLog
Sistemde kim ne yaptı kaydı.

| Alan                    | Tip              | Açıklama |
|-------------------------|------------------|----------|
| id                      | UUID (PK)        | - |
| actor_type              | VARCHAR(50)      | admin, system, tool |
| actor_id                | UUID             | - |
| action                  | VARCHAR(100)     | tool.created, rule.updated... |
| resource_type           | VARCHAR(50)      | - |
| resource_id             | UUID             | - |
| details                 | JSONB            | - |
| ip_address              | INET             | - |
| created_at              | TIMESTAMPTZ      | - |

### 8. AdminUser (ek entity – auth için gerekli)
| Alan                    | Tip              | Açıklama |
|-------------------------|------------------|----------|
| id                      | UUID (PK)        | - |
| email                   | VARCHAR(255)     | Unique |
| password_hash           | VARCHAR(255)     | argon2id |
| full_name               | VARCHAR(150)     | - |
| is_active               | BOOLEAN          | - |
| created_at              | TIMESTAMPTZ      | - |
