# Terrova / Harmanım - Mimari Genel Bakış

## Ürün ve Hedef Kullanıcı

Terrova (Harmanım), Türkiye'nin tarım ekosistemi için kurulmuş bir pazaryeridir.
Dört farklı kullanıcı grubunu bir araya getirir:

- **Çiftçiler** — arazi kiralayabilir, ekipman bulabilir, işçi arayabilir
- **Arazi sahipleri** — arazilerini ilan edebilir
- **Tarım işçileri** — iş ilanlarına başvurabilir
- **Ekipman kiralayanlar** — makine/ekipmanlarını kiraya verebilir

Amaç, bu dört grubu tek bir platformda buluşturup tarımsal kaynakların
(arazi, işgücü, ekipman) daha verimli paylaşılmasını sağlamak.

## Bileşenler ve Sorumlulukları

Sistem üç ana bileşenden oluşur:

### 1. Mobil Uygulama (terrova-mobil)

- **Teknoloji:** React Native (Expo), iOS + Android
- **Sorumluluk:** Son kullanıcının (çiftçi, arazi sahibi, işçi, kiralayan)
  uygulamayla asıl etkileşim kurduğu yer. Kayıt/giriş, ilan görüntüleme,
  ilan oluşturma, arazi detay ve "Field Log" (arazi günlüğü) gibi
  özellikler burada.
- **Ana ekranlar:** 5 sekmeli ana navigasyon, auth (OTP/email/kayıt),
  listing (ilan detay + oluşturma), land (arazi detay + Field Log)
- Tüm verileri terrova-api'den çeker, kendi başına veri saklamaz.

### 2. Backend API (terrova-api)

- **Teknoloji:** NestJS, PostgreSQL (Prisma ORM)
- **Sorumluluk:** Sistemin "beyni". Tüm veri, kimlik doğrulama ve iş
  mantığı burada yönetilir. Mobil ve Web, tüm işlemler için bu API'ye
  istek atar.
- **Kimlik doğrulama:** Telefon numarası + OTP (SMS kodu) ile giriş,
  JWT token ile oturum yönetimi, refresh token ile oturum yenileme.
- **Diğer:** Standart hata formatı, istek sınırlama (rate limiting),
  versiyonlanmış uç noktalar (`/v1`)

### 3. Web / Landing (terrova-landing)

- **Teknoloji:** Next.js, TypeScript, Tailwind CSS
- **Sorumluluk:** Terrova'nın tanıtım/pazarlama sitesi. Şu an için
  uygulamanın kendisini değil, ürünü tanıtan genel bir web sayfası.
- **Deploy:** Vercel üzerinden yayınlanıyor.

## Temel İlan Akışı (Üst Düzey Özet)

Örnek bir "ilan oluşturma" akışı şu şekilde işler:

1. Kullanıcı mobil uygulamada telefon numarasıyla giriş yapar
   (OTP kodu SMS ile gelir, doğrulanır → terrova-api bir JWT token döner)
2. Kullanıcı mobil uygulamada "ilan oluştur" ekranına girer, bilgileri doldurur
3. Mobil uygulama bu bilgileri terrova-api'ye gönderir (JWT token ile birlikte)
4. terrova-api, isteği doğrular, veritabanına (PostgreSQL) kaydeder
5. İlan artık sistemde yer alır; diğer kullanıcılar mobil uygulamadan
   bu ilanı görüntüleyip başvurabilir/iletişime geçebilir

Bu akışta Web (terrova-landing) herhangi bir rol oynamaz — sadece
tanıtım amaçlıdır, gerçek ilan/işlem akışına dahil değildir.
