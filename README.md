# Yakınımda — başlangıç proje paketi

Yakınımda; yerel topluluk, ilan pazarı, işletme keşfi, iş ilanları ve sosyal içerikleri bir araya getiren Android-öncelikli bir platformdur.

Bu paket **çalışan, üretime hazır bir APK değildir**. Flutter uygulama iskeleti, ilk veritabanı şeması, güvenlik politikaları ve geliştirme planı içerir. Gerçek kullanıcı kabulü için Supabase projesi, OAuth/SMS sağlayıcıları, uygulama imzalama, testler, hukuki metinler ve mağaza hazırlığı tamamlanmalıdır.

## Ürün kararları

- Dünya genelinden üyelik.
- E-posta/şifre, Google OAuth ve telefon/SMS OTP ile giriş.
- Kullanıcıların konum izni vermeden de ülke/şehir/ilçe seçerek kullanabilmesi.
- Normal ilanlar otomatik denetimden geçince yayınlanır.
- Şüpheli, yasaklı veya yüksek riskli ilanlar otomatik incelemeye alınır.
- Yönetici ilanı düzenleyebilir, yayından kaldırabilir, reddedebilir; kullanıcıyı askıya alabilir/engelleyebilir.
- IP adresleri yalnızca güvenlik ve kötüye kullanım önleme amacıyla sunucu tarafında kaydedilir. Yönetici erişimi sınırlandırılır, erişimler denetlenir ve saklama süresi hukuki değerlendirmeyle yapılandırılır.
- Kullanıcıların açık IP adresleri herkese veya diğer kullanıcılara gösterilmez.

## Teknoloji önerisi

- Flutter: Android ilk sürüm; aynı kod tabanıyla ileride iOS.
- Supabase Auth + PostgreSQL + Storage + Edge Functions.
- Firebase Cloud Messaging: push bildirimleri.
- Harita sağlayıcısı: ülke ve maliyet gereksinimlerine göre seçilecek.
- Web yönetim paneli: ayrı bir yönetici uygulaması.

## Başlatma

Flutter SDK ve Android Studio kurulu olmalıdır.

```bash
cd flutter_app
flutter pub get
flutter run
```

Supabase bağlantısı için `flutter_app/lib/config.dart` içindeki yer tutucuları kendi Supabase URL ve anon key'inizle değiştirin. Anon key istemci uygulamasında kullanılabilir; **service_role key asla uygulamaya konulmamalıdır**.

Veritabanı migration dosyasını Supabase SQL Editor veya Supabase CLI ile uygulayın.

## Kimlik doğrulama

Supabase Dashboard'da:
1. Email provider'ı etkinleştirin.
2. Google OAuth provider'ını yapılandırın.
3. Phone provider ve SMS sağlayıcısını yapılandırın.
4. Ülke kodları, OTP limitleri, CAPTCHA ve rate limit ayarlarını test edin.
5. Redirect URL'leri Android uygulama kimliğinizle eşleştirin.

SMS ve telefon doğrulama ücretleri sağlayıcıya ve ülkeye göre değişebilir. Google girişinin de OAuth istemci kimlikleri gerektirir.

## Otomatik ilan denetimi

Bu pakette durum alanları ve temel SQL akışı tanımlanmıştır. Üretimde otomatik denetim; sunucu tarafı Edge Function/iş kuyruğu üzerinden yapılmalıdır. İstemci uygulamasının kendi ilanını `published` durumuna getirmesine izin verilmez.

Önerilen kararlar:
- `approved`: otomatik kontrolleri geçen ilan; yayınlanır.
- `review`: belirsiz/şüpheli içerik; yönetici kuyruğuna gider.
- `rejected`: açıkça yasak veya zararlı içerik; yayınlanmaz, gerekçesi kaydedilir.
- `removed`: daha önce yayınlanmış ilan yönetici tarafından kaldırılmıştır.

Otomatik sınıflandırma tek başına hukuki karar mekanizması değildir. Özellikle silah, uyuşturucu, sahte ürün, finansal dolandırıcılık, yetişkin içerik ve tehlikeli hizmet kategorileri için açık kurallar ve insan incelemesi gerekir.

## IP ve gizlilik

IP adresi kişisel veri niteliği taşıyabilir. Bu nedenle:
- IP yalnızca sunucu tarafında, güvenilir proxy başlıkları doğru yapılandırılarak alınmalıdır.
- Kullanıcıdan gelen `X-Forwarded-For` benzeri başlıklar körü körüne güvenilir kabul edilmemelidir.
- IP görüntüleme yalnızca yetkili güvenlik yöneticileriyle sınırlandırılmalıdır.
- Yönetici erişimleri ayrıca loglanmalıdır.
- Saklama süresi, amaç, aydınlatma metni ve olası yurt dışı aktarım KVKK/GDPR uzmanıyla belirlenmelidir.
- Bu örnek migration IP için hash alanı kullanır; ham IP saklama gereksinimi ayrıca hukuki ve güvenlik incelemesiyle kararlaştırılmalıdır.

## Sonraki geliştirme adımları

1. Marka/domain ve uygulama paket kimliği kararı.
2. Flutter ekranlarının tamamlanması.
3. Supabase projesi ve gerçek auth bağlantıları.
4. İlan oluşturma, fotoğraf yükleme, arama ve filtreler.
5. Sunucu tarafı otomatik moderasyon kuyruğu.
6. Yönetici web paneli ve denetim kayıtları.
7. Mesajlaşma, harita, işletme profilleri ve bildirimler.
8. Güvenlik, performans, KVKK/GDPR ve mağaza testleri.


## Üye paneli, mesajlaşma ve favoriler

`supabase/migrations/002_member_features.sql` konuşmalar, mesajlar, favoriler, bildirimler ve kullanıcı şikâyetleri tablolarını ekler. `docs/MEMBER_PANEL.md` üye paneli gereksinimlerini açıklar. Mesaj gönderme ve konuşma başlatma, engelleme/rate-limit kontrolleri nedeniyle güvenilir sunucu fonksiyonları üzerinden tamamlanmalıdır.
