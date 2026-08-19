# 02 - Service Architecture

## Servis Listesi ve Sorumlulukları

| Servis              |              Sorumluluk                                              | İletişim Türü          |
|---------------------|----------------------------------------------------------------------|------------------------|
| Admin API           | Tool, EventType, Subscription, RoutingRule ve API key yönetimi      | HTTP (REST)            |
| Event Ingest        | Publisher’lardan gelen event’leri doğrular, persist eder ve Kafka’ya yazar | HTTP + Kafka           |
| Routing Engine      | Kafka’dan event okur, subscription ve rule’lara göre delivery task üretir | Kafka                  |
| Delivery Worker     | Delivery task’ları alır, webhook atar, retry ve DLQ yönetir         | Kafka + HTTP           |
| DLQ Handler         | Ölü mesajları saklar, admin’e sunar ve replay imkânı verir          | Kafka + HTTP           |
| Admin UI            | Platformun yönetim arayüzü                                           | HTTP                   |
| Mock Publisher      | Test amaçlı event üretir                                             | HTTP                   |
| Mock Subscriber     | Test amaçlı webhook alır                                             | HTTP                   |
| Webhook Sink        | Debug amaçlı tüm webhook’ları yakalar ve gösterir                    | HTTP                   |

## Neden Bu Şekilde Bölündü?

- Single Responsibility: Her servis tek bir işe odaklanır. Örnek: Ingest sadece event kabul eder, routing kararını vermez.
- Bağımsız ölçeklenebilirlik: Delivery Worker yavaş subscriber’lardan dolayı darboğaz yaşarsa sadece o servis scale edilir.
- Hata izolasyonu: Delivery Worker çökerse Ingest ve Routing çalışmaya devam eder.
- Geliştirme kolaylığı: Farklı kişiler farklı servisler üzerinde paralel çalışabilir.

## Senkron vs Asenkron Kararları

- Senkron (HTTP): Admin API, Event Ingest endpoint’i, Admin UI ile backend arası.
- Asenkron (Kafka): Event Ingest → Routing Engine → Delivery Worker akışı. Çünkü event’lerin kaybolmaması, retry ve yüksek hacim için gereklidir.

## Alternatif Bölünmeler ve Dezavantajları

- Tek monolit: Tüm mantık tek uygulamada → scale edilemez, hata tüm sistemi etkiler.
- Ingest + Routing birleştirme: Routing yavaşladığında event kabulü de yavaşlar.
- Delivery Worker’ı Routing’e gömme: Retry ve circuit breaker karmaşıklığı routing’i kirletir.
