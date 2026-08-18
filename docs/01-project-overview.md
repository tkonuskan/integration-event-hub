# 01 - Project Overview

## Sistemin Amacı

Integration & Event Hub, bağımsız yazılım ürünlerinin birbirleriyle event (olay) tabanlı ve gevşek bağlı şekilde haberleşmesini sağlayan merkezi bir platformdur. 

Platform, publisher tool’ların fırlattığı event’leri alır, routing kurallarına göre ilgili subscriber tool’lara güvenilir şekilde iletir, hata durumlarında retry yapar, başarısız mesajları Dead Letter Queue’ya (DLQ) taşır ve tüm süreci gözlemler.

Temel hedef, hiçbir tool’un diğerine hardcoded şekilde bağlı olmadığı, yeni tool’ların kolayca eklenebildiği ve mevcut tool’ların etkilenmeden çalışmaya devam ettiği bir PaaS tarzı merkezi ve konfigürasyon tabanlı bir entegrasyon katmanı oluşturmaktır.

## Kimler Kullanır? (Aktörler)

| Aktör              | Rolü                                                                 | Sisteme Ne Verir?                          | Sistemden Ne Alır?                              |
|--------------------|----------------------------------------------------------------------|--------------------------------------------|-------------------------------------------------|
| Admin              | Platformu yöneten kişi                                               | Tool kaydı, event type, subscription, routing rule, API key | Dashboard, loglar, DLQ yönetimi, audit log |
| Publisher Tool     | Event üreten bağımsız uygulama                                       | Event (API key + HMAC ile)                 | Event kabul onayı / hata mesajı                 |
| Subscriber Tool    | Event’e abone olan bağımsız uygulama                                 | Webhook endpoint bilgisi, subscription     | HTTP webhook (HMAC imzalı)                      |
| Platform Servisleri| Veri Alma, Yönlendirme, Teslimat, DLQ Handler vb.                           | -                                          | Birbirleriyle Apache Kafka + HTTP üzerinden haberleşir        |
| Mock Tool’lar      | Geliştirme ve test amaçlı simülasyon araçları                        | Test event’leri / test webhook’ları        | Platformun davranışını doğrulama imkânı         |

-  Admin: Tüm CRUD (Create, Rename, Update, Delete) işlemlerini yapar, sistemi izler, DLQ’dan mesaj replay eder.
-  Publisher Tool: Sadece event fırlatır. Kimlik kanıtı için API key + HMAC kullanır.
-  Subscriber Tool: Webhook endpoint’i açar, gelen istekleri HMAC ile doğrular.
-  Platform içi servisler: Birbirleriyle senkron HTTP veya asenkron Kafka ile konuşur.

## Out of Scope (Sistemin Yapmadığı Şeyler)

Bu platform, yalnızca araçlar arasındaki iletişimi ve birlikte çalışmayı sağlar

Aşağıdakiler kapsam dışındadır:

- Tool’ların kendi kendi iç iş mantıklarını (business logic’ini) çalıştırmak
- Belge üretme, regülasyon kontrolü, kullanıcı yönetimi gibi domain işleri
- Tool’ların kendi veritabanlarını veya kalıcı depolarını yönetmek
- Son kullanıcıya yönelik arayüz sağlamak (sadece Admin UI vardır)
- Ödeme, faturalandırma, abonelik yönetimi
- Gerçek zamanlı mesajlaşma veya chat sistemi
- Tool’ların içindeki authentication/authorization mantığı

## Tak-Çıkart (Plug-and-Play) Mantığı

Yeni bir tool ekosisteme eklendiğinde:

1. Admin paneli veya Admin API üzerinden tool kaydı yapılır.
2. Platform tool’a bir API key üretir (hash’lenerek saklanır).
3. Tool, yayınlayacağı event type’ları tanımlar.
4. İlgili subscriber’lar subscription ve routing rule oluşturur.

Bu işlemler sırasında hiçbir mevcut servisin koduna dokunulmaz. Tüm bağlantılar dinamik ve konfigürasyon tabanlıdır. 

Bir tool sistemden çıkarıldığında da diğer tool’lar çalışmaya devam eder çünkü aralarında doğrudan bağımlılık yoktur.
