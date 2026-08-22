# 06 - Kafka Topic Tasarımı

## Topic Listesi

| Topic                  | Amaç                              | Partition | Retention | Not |
|------------------------|-----------------------------------|-----------|-----------|-----|
| events.ingested        | Event Ingest’ten çıkan ham event  | 6         | 7 gün     | Ana giriş |
| events.delivery        | Routing Engine’in ürettiği task’lar | 12      | 3 gün     | Delivery Worker dinler |
| events.dlq             | Max retry aşmış mesajlar          | 3         | 30 gün    | DLQ Handler dinler |
| cache.invalidate       | Rule/subscription değişince       | 3         | 1 gün     | Broadcast |

## Partition Sayısı Kararı
- `events.ingested`: 6 → orta seviye parallelism
- `events.delivery`: 12 → Delivery Worker daha fazla scale edilebilsin diye
- Sayılar environment variable ile konfigüre edilebilir.

## Ordering Garantisi
- Aynı `event_id` veya aynı `source_tool_id` için sıralama istenirse partition key olarak `source_tool_id` veya `event_id` kullanılır.
- Genel durumda ordering zorunlu değildir.

## Consumer Group Stratejisi
- Routing Engine → `routing-engine-group`
- Delivery Worker → `delivery-worker-group`
- DLQ Handler → `dlq-handler-group`

Aynı group içindeki instance’lar partition’ları paylaşır (horizontal scaling).

## Dead Letter Queue
- Max retry (örnek: 5) aşıldığında mesaj `events.dlq` topic’ine atılır.
- Orijinal event + son hata bilgisi + attempt sayısı payload’da tutulur.
- Admin panelden manuel replay yapılabilir.

## Auto Topic Creation
Kapalı olacak. Init container ile topic’ler oluşturulacak (production best practice).
