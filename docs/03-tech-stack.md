# 03 - Tech Stack Kararları

## Backend Dili ve Framework

Değerlendirilen alternatifler:
1. **Go + Chi/Echo** → Yüksek performans ve düşük kaynak kullanımı. Ancak öğrenme eğrisi ve geliştirme hızı staj süresi için riskli.
2. **Node.js + NestJS/Fastify** → Hızlı geliştirme ve TypeScript avantajı var. Ancak Python’a göre biraz daha boilerplate.
3. **Python + FastAPI** → En hızlı geliştirme, yerleşik OpenAPI, yüksek okunabilirlik, async desteği yeterli.

**Seçim: Python + FastAPI**

**Gerekçe:**  
Projenin kapsamı (çok sayıda servis + Admin UI + observability + test) göz önüne alındığında en dengeli seçimdir. Contract-first yaklaşım FastAPI ile doğal şekilde desteklenir. Performans kritik noktalarda (Delivery Worker) async + httpx ile yönetilebilir seviyededir.

## Frontend

1. React + Vite
2. Next.js
3. Vue 3 + Vite

**Seçim: React + Vite + TypeScript**

**Gerekçe:**  Admin UI için server-side rendering (SSR) veya SEO (Arama Motoru Optimizasyonu) gerekmiyor. Vite’ın hızı ve React’in geniş ekosistemi, staj süresinde arayüzü hazır component kütüphaneleriyle hızlıca ayağa kaldırmayı sağlar.

## Veritabanı ve ORM

- PostgreSQL
- Prisma Client Python 

**Gerekçe:** SQLAlchemy, kurulumu ve migration (Alembic) yönetimi karmaşık olabilir ve zaman kaybettirebilir. Prisma Client Python, tüm veritabanı şemasını tek bir okunabilir dosyada (schema.prisma) toplamamızı, otomatik type-safe (tip güvenli) kod üretmemizi ve prisma migrate komutlarıyla veritabanı güncellemelerini saniyeler içinde halletmemizi sağlayarak geliştirme hızımızı maksimuma çıkaracaktır.

## Diğer Kritik Kütüphaneler

| Amaç                    | Seçilen Kütüphane          | Neden? |
|-------------------------|---------------------------|------|
| Kafka                   | aiokafka / confluent-kafka | Async uyumlu |
| Redis                   | redis (asyncio)           | Rate limit, cache, lock |
| HTTP Client (webhook)   | httpx                     | Async + timeout kontrolü kolay |
| Auth (JWT + password)   | python-jose + passlib     | Argon2 desteği var |
| Validation              | Pydantic                  | FastAPI ile entegre |
| Testing                 | pytest + httpx            | Integration test için uygun |

## Servisler Arası İletişim

- **REST + OpenAPI** (Admin API, Event Ingest,Webhook gönderimleri)
- **Kafka** (event akışı)
