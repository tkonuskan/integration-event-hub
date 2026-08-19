# Integration & Event Hub

Geliştirilen proje, bağımsız yazılımların (tool’ların) birbirleriyle event tabanlı ve gevşek bağlı şekilde haberleşmesini sağlayan merkezi Integration & Event Hub platformudur.

## Proje Amacı

Platform, publisher tool’ların fırlattığı event’leri alır, tanımlı routing kurallarına göre ilgili subscriber tool’lara güvenilir şekilde iletir. Retry, Dead Letter Queue (DLQ), idempotency, rate limiting ve gözlemleme gibi dağıtık sistem ihtiyaçlarını karşılar.

Temel prensip: Hiçbir tool diğerine hardcoded bağlı değildir. Yeni tool eklemek veya çıkarmak mevcut sistemi bozmaz (tak-çıkart / plug-and-play mimari).

## Mevcut Durum

| Adım | Durum | Açıklama |
|------|------|----------|
| Adım 1 | Tamamlandı | Proje kapsamı, aktör analizi, microservice mimarisi, teknoloji seçimi |
| Adım 2 | Devam Ediyor | Domain model, ER tasarımı, Postgres / Redis / Kafka stratejileri |


## Teknoloji Stack

**Backend:** Python + FastAPI  
**Frontend:** React + Vite + TypeScript  
**Veritabanı:** PostgreSQL 
**ORM:** SQLModel + Alembic
**Cache / Rate Limit / Lock:** Redis  
**Event Streaming:** Apache Kafka  
**Containerization:** Docker + Docker Compose  


## Servisler

| Servis | Sorumluluk |
|--------|----------|
| Admin API | Tool, EventType, Subscription, RoutingRule ve API Key yönetimi |
| Event Ingest | Publisher’lardan event kabul etme, doğrulama, persist ve Kafka’ya yazma |
| Routing Engine | Event’leri okuyup subscription + rule’lara göre delivery task üretme |
| Delivery Worker | Webhook teslimatı, retry, circuit breaker, DLQ’ya gönderme |
| DLQ Handler | Ölü mesajların yönetimi ve replay |
| Admin UI | Yönetim arayüzü |

## Dokümantasyon

Tüm tasarım kararları ve gerekçeler `docs/` klasöründe tutulmaktadır. Her mimari karar yazılı olarak gerekçelendirilmiştir.

## Çalıştırma (Sonraki adımlarda aktif olacak)

