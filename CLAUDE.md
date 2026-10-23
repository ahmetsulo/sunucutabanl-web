Multi-Tenant Ekip Yetkinlik Takip SaaS - AI Geliştirici Talimatları (CLAUDE.md)

Bu dosya, bu projede çalışırken AI asistanının (Claude, Cursor vb.) uyması gereken mimari kuralları, teknoloji standartlarını ve geliştirme ilkelerini tanımlar.

1. Teknoloji Stack'i

Framework: Next.js (App Router, React Server Components)

Dil: TypeScript (Strict Mode)

Veritabanı: PostgreSQL

ORM: Prisma ORM

Gerçek Zamanlı İletişim: Server-Sent Events (SSE)

Stil / UI: Tailwind CSS, Shadcn UI / Radix Primitives

2. Temel Mimari Kurallar ve Multi-Tenancy (Çoklu Kiracılık)

Tenant İzolasyonu (KRİTİK):

Veritabanındaki her bir tabloda (sistem düzeyindeki global meta veriler hariç) tenant_id alanı bulunması zorunludur.

Tüm Prisma sorgularında (findMany, findFirst, update, delete vb.) where koşuluna tenant_id eklenmelidir.

Kullanıcının istek attığı tenant dışındaki bir veriye erişmesine kesinlikle izin verilmemelidir (Row-Level / Application-Level Security).

Next.js App Router Standartları:

Veri çekme işlemleri mümkün olduğunca Server Components (RSC) içinde yapılmalıdır.

İstemci taraflı etkileşim gerektiren bileşenlerin en üstüne 'use client' direktifi eklenmelidir.

Form işlemleri ve mutasyonlar için Server Actions kullanılmalıdır.

Gerçek Zamanlı Güncellemeler (SSE):

Ekip yetkinlik güncellemeleri ve canlı bildirimler için WebSocket yerine Server-Sent Events (SSE) mimarisi kullanılmalıdır.

SSE route'ları Next.js Route Handlers (app/api/.../route.ts) içerisinde ReadableStream yapısıyla kurgulanmalıdır.

3. Güvenlik ve Gizlilik Kuralları

Hassas Bilgiler (Secrets):

Kod içerisine kesinlikle hiçbir API Key, Secret, Connection String veya parola hardcode YAZILAMAZ.

Tüm hassas veriler .env.local veya ortam değişkenleri (Environment Variables) üzerinden process.env.VARIABLE_NAME şeklinde okunmalıdır.

Veri Doğrulama ve Güvenlik:

Server Action veya API Route Handler'a gelen tüm girdiler Zod veya benzer bir şema doğrulayıcı ile validate edilmelidir.

Yetkilendirme (RBAC) her Server Action ve API çağrısının başında kontrol edilmelidir.

4. Dil ve Dil Mimarisi

Arayüz Dili: Kullanıcı arayüzü (UI) metinleri, butonlar, hata mesajları ve bildirimler tamamen Türkçe olacaktır.

Kod Kodlaması ve Açıklamalar:

Kod içi yorum satırları (comments) ve JSDoc açıklamaları Türkçe yazılmalıdır.

Değişken isimleri, fonksiyonlar, tip tanımları ve dosya adları İngilizce (clean code / industry standard) yazılmalıdır (Örn: getUserSkillLevel, TenantMiddleware).

5. Kod Formatı ve Tip Güvenliği

any tipi kullanımı kesinlikle yasaktır. Mümkün olan her yerde spesifik TypeScript tipleri veya unknown kullanılmalıdır.

Prisma modelleri güncellendiğinde type-safe client üretimi (npx prisma generate) göz önünde bulundurulmalıdır.

Modüler, sürdürülebilir ve clean code prensiplerine uygun, DRY (Don't Repeat Yourself) ilkelerini benimseyen kodlar üretilmelidir.
