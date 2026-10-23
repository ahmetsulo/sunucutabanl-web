Mimari Karar Kaydı (Architecture Decision Record - ADR)

Proje: Kurumsal Yetkinlik ve Bildirim Yönetim Sistemi

Tarih: 23 Ekim 2026

Durum: Kabul Edildi (Accepted)

1. Servis Modeli: PaaS (Vercel) ve Yönetilen PostgreSQL

Kontekst

Uygulamanın altyapı kurulum ve bakım maliyetlerini (DevOps yükü) en aza indirirken; hızlı yaygınlaştırma, otomatik ölçeklenme ve yüksek erişilebilirlik sağlayan bir altyapı mimarisine ihtiyaç duyulmaktadır.

Karar

Platform dağıtımı için PaaS (Platform as a Service) olarak Vercel, ilişkisel veritabanı ihtiyacı için ise yönetilen PostgreSQL (ör. Supabase / Neon / Vercel Postgres) kullanılmasına karar verilmiştir.

Gerekçe ve Değerlendirme

Düşük Sunucu/DevOps Yönetimi: PaaS modeli sayesinde sunucu güncellemesi, işletim sistemi yamaları veya yük dengeleyici (load balancer) yapılandırmasıyla vakit kaybetmeden doğrudan ürün geliştirmeye odaklanılabilir.

Güçlü İlişkisel Veri Yapısı: Yetkinlik matrisleri, kullanıcı rolleri, şirket/tenant ilişkileri ve karmaşık sorgular için ACID garantisi sunan ilişkisel bir veritabanı (PostgreSQL) gereklidir.

Sürekli Entegrasyon & Yaygınlaştırma (CI/CD): Vercel'in GitHub entegrasyonu sayesinde her commit/PR için otomatik önizleme ve canlıya alma süreçleri zahmetsizce yönetilir.

Alternatifler

IaaS (AWS EC2 / DigitalOcean Droplet): Tam kontrol sağlasa da sunucu yönetimi, güvenlik güncellemeleri ve manuel ölçeklendirme maliyeti yüksektir.

NoSQL (MongoDB): Esnek şema sağlasa da katı ilişkisel veriler ve veri bütünlüğü (ACID) gereksinimleri için PostgreSQL kadar güçlü garantiler sunmaz.

Sonuçlar

Olumlu: Hızlı canlıya çıkış süresi (Time-to-Market), sıfır altyapı bakım maliyeti, otomatik HTTPS/SSL ve CDN desteği.

Olumsuz: Vercel sağlayıcısına bağımlılık (Vendor Lock-in) riski; ancak kod standart kaldığı sürece başka bir ortama (ör. Docker/Container) taşınabilir.

2. Barındırma & Mimari: Serverless API Rotaları

Kontekst

Sistemin trafik profili mesai saatleri içinde yoğun, mesai saatleri dışında ve hafta sonlarında düşüktür. Sürekli çalışan Monolitik veya Dedicated sunucular boşta kaldığı süre boyunca gereksiz maliyet yaratmaktadır.

Karar

Backend mimarisi için Serverless (Olay Güdümlü / Event-Driven) API Rotaları (Next.js API Routes / Vercel Serverless Functions) kullanılmasına karar verilmiştir.

Gerekçe ve Değerlendirme

Maliyet Verimliliği (Pay-as-you-go): Sunucular yalnızca istek (request) geldiğinde çalışır ve saniye/milisaniye bazında faturalandırılır. Trafik olmadığında maliyet sıfıra yakındır.

Otomatik Ölçeklenme: Anlık yük artışlarında (örneğin tüm şirket çalışanlarına aynı anda bildirim gittiğinde) sistem otomatik olarak yatayda ölçeklenir.

Modülerlik: Her API rotası bağımsız bir fonksiyon olarak çalıştığı için sistemin bir bölümündeki hata diğer bölümlerin çalışmasını engellemez.

Alternatifler

Geleneksel Express.js / Node.js Monolit Sunucu: 7/24 çalışması gerektiği için yüksek kararlı sunucu maliyeti çıkarır ve anlık yük artışlarında manuel ölçeklendirme gerektirir.

Sonuçlar

Olumlu: Sınırsız otomatik ölçeklenme, kullanım kadar ödeme modeli.

Olumsuz: Soğuk Başlangıç (Cold Start) gecikmeleri yaşanabilir. Veritabanı bağlantı havuzu (Connection Pooling) yönetimi dikkatli kurgulanmalıdır (ör. Prisma Accelerate, PgBouncer).

3. Kiracı (Tenant) Yapısı: Ortak Veritabanı, Ortak Tablolar ve tenant_id İle Veri Ayrımı

Kontekst

Sistem Çoklu Kiracılı (Multi-tenant) yapıda çalışacaktır. Farklı şirketlerin/organizasyonların verilerinin mantıksal olarak birbirinden tamamen izole edilmesi gerekmektedir. Aynı zamanda veritabanı maliyetlerinin ve şema güncellemelerinin (migration) kolay yönetilebilir olması kritik bir gereksinimdir.

Karar

Multi-tenancy mimarisinde "Ortak Veritabanı, Ortak Tablolar" (Shared Database, Shared Schema) yaklaşımı seçilmiş ve tüm kiracıya özel tablolara zorunlu tenant_id kolonu eklenerek veri ayrımı sağlanmıştır.

Gerekçe ve Değerlendirme

Maliyet ve Kaynak Verimliliği: Her kiracı için ayrı veritabanı veya ayrı şema açmak veritabanı bağlantı sınırlarını zorlar ve maliyeti katlar. Ortak veritabanı en ekonomik modeldir.

Bakım Kolaylığı: Şema güncellemeleri (Database Migrations) tek bir veritabanı üzerinde çalıştırılarak tüm kiracılar için aynı anda güncellenmiş olur.

Güvenlik Mekanizması: ORM (ör. Prisma, Drizzle) veya veritabanı seviyesinde (PostgreSQL Row Level Security - RLS) uygulanacak politikalar ile tenant_id filtresi zorunlu kılınarak veri sızıntısı engellenir.

Alternatifler

Database-per-Tenant (Her Kiracıya Ayrı Veritabanı): En yüksek izolasyonu sağlar ancak yüzlerce kiracıda bakım ve maliyet yönetimi imkansız hale gelir.

Schema-per-Tenant (Her Kiracıya Ayrı Şema): Şema güncellemelerini karmaşıklaştırır ve veritabanı bağlantı havuzunu hızlıca tüketir.

Sonuçlar

Olumlu: Düşük veritabanı maliyeti, basitleştirilmiş veri tabanı yönetimi ve kolay veri analitiği.

Olumsuz: tenant_id filtresinin unutulması durumunda veri sızıntı riski (Sorgu katmanında Middleware veya PostgreSQL RLS ile bu risk sıfırlanmalıdır).

4. Gerçek Zamanlı İletişim: Kritik Yetkinlik Uyarısı İçin SSE (Server-Sent Events)

Kontekst

Sistemde kritikliğe sahip bir yetkinlik açığı veya acil durum uyarısı oluştuğunda, kullanıcı arayüzüne (UI) anlık ve gerçek zamanlı bildirim düşmesi gerekmektedir.

Karar

Kullanıcılara anlık bildirim iletimi için SSE (Server-Sent Events) teknolojisinin kullanılmasına karar verilmiştir.

Gerekçe ve Değerlendirme

Tek Yönlü İletişim İhtiyacı: Kritik yetkinlik uyarıları yalnızca sunucudan istemciye (Server-to-Client) doğru akar. İstemcinin sürekli sunucuya veri göndermesine gerek yoktur. SSE bu senaryo için biçilmiş kaftandır.

HTTP/HTTPS Protokol Uyumluluğu: SSE standart HTTP protokolü üzerinden çalışır. Güvenlik duvarları (Firewall), proxy veya yük dengeleyiciler tarafından engellenme riski yoktur.

Düşük Kaynak Tüketimi: HTTP Long-Polling gibi sürekli sunucuyu yormaz; bağlantı bir kez açılır ve sunucu olay geldikçe veriyi iter. Otomatik yeniden bağlanma (reconnection) desteği tarayıcı tarafından varsayılan olarak sunulur.

Alternatifler

WebSockets: Çift yönlü (Bi-directional) iletişim sağlar ancak durum bilgisi tutan (stateful) yapısı nedeniyle Serverless ortamlarla entegrasyonu zordur ve ek bir aracı servis (Pusher, Ably vb.) gerektirir.

HTTP Polling (Periyodik İstek): Sunucuya gereksiz yük bindirir, gecikmeli veri aktarımına yol açar ve verimsizdir.

Sonuçlar

Olumlu: Basit kurulum, varsayılan tarayıcı desteği (EventSource API), Serverless uyumluluğu ve düşük bant genişliği kullanımı.

Olumsuz: İstemciden sunucuya anlık veri gönderme ihtiyacı doğarsa tek başına yetersiz kalır (bu durumda standart REST API endpoint'leri ile desteklenir).
