# 🔐 Silencia – Güvenlik Politikası

Bu doküman, **Silencia mobil uygulamasında** kullanıcı verilerinin korunması,
kimlik doğrulama süreçleri ve sistem güvenliğine yönelik alınan önlemleri
açıklamak amacıyla hazırlanmıştır.

---

## 🎯 Güvenlik Yaklaşımı

Silencia, **privacy-by-design** ve **secure-by-default** prensipleri ile
geliştirilmiştir.

Temel hedefler:
- Kullanıcı verilerinin gizliliğini korumak
- Yetkisiz erişimleri engellemek
- Veri bütünlüğünü sağlamak
- Güvenli iletişim ve servisler arası izolasyon

---

## 🔐 Kimlik Doğrulama (Authentication)

### Kullanılan Sistem
- **Firebase Authentication**

### Desteklenen Yöntemler
- Email / Password
- Google Sign-In

### Güvenlik Önlemleri
- Şifreler **plain text olarak saklanmaz**
- Hash + salt işlemleri Firebase tarafından yönetilir
- Oturum bilgileri güvenli şekilde tutulur
- Yetkisiz oturum açma denemeleri Firebase tarafından sınırlandırılır

---

## 👤 Yetkilendirme (Authorization)

Silencia’da yetkilendirme, **kullanıcı bazlı erişim kontrolü** ile sağlanır.

- Her kullanıcı yalnızca **kendi verilerine** erişebilir
- Arkadaş listesi ve mesajlaşma erişimleri karşılıklı izin esasına dayanır
- Admin yetkileri yalnızca yetkili hesaplara tanımlıdır

---

## 🗄️ Veri Güvenliği

### Firestore Güvenliği
- **Firestore Security Rules** aktif olarak kullanılır
- Kullanıcı verileri `userId` bazlı izole edilir
- Yetkisiz okuma / yazma işlemleri engellenir

### Örnek Güvenlik Prensibi
- Kullanıcı yalnızca:
  - Kendi profilini
  - Kendi favorilerini
  - Kendi istatistiklerini
  düzenleyebilir

---

## 💬 Mesajlaşma Güvenliği

- Mesajlar yalnızca ilgili iki kullanıcı tarafından erişilebilir
- Firestore kuralları ile sohbet verileri korunur
- Sohbet verileri dış kullanıcılar tarafından okunamaz

---

## 📷 Kamera ve Medya Güvenliği

- Kamera erişimi yalnızca kullanıcı izni ile sağlanır
- Kamera görüntüleri:
  - Kalıcı olarak saklanmaz
  - Sadece anlık işaret dili analizi için kullanılır
- Medya dosyaları Firebase Storage üzerinde güvenli kurallarla tutulur

---

## 🌐 Ağ Güvenliği

- Tüm istemci – backend iletişimi **HTTPS** üzerinden yapılır
- API çağrıları güvenli protokoller ile gerçekleştirilir
- Ortadaki adam (MITM) saldırılarına karşı şifreli iletişim kullanılır

---

## 🧠 Makine Öğrenmesi (ML) Servisi Güvenliği

- İşaret dili tanıma sistemi **ayrı ve izole bir servis** olarak çalışır
- ML servisi:
  - Public erişime açık değildir
  - Sadece backend tarafından çağrılır
- Model dosyaları ve eğitim verileri **private repository** içinde tutulur

> 🔒 Not: ML model kodları ve eğitim verileri güvenlik ve gizlilik
> sebebiyle public olarak paylaşılmamaktadır.

---

## 📦 Local Veri Güvenliği

- Yerel cihazda tutulan veriler:
  - Kullanıcı tercihleri
  - Tema ayarları
  - Cache verileridir
- Hassas kullanıcı bilgileri cihazda **şifrelenmeden saklanmaz**

---

## 🧪 Test ve Güvenlik Kontrolleri

- Auth akışları manuel olarak test edilmiştir
- Yetkisiz erişim senaryoları kontrol edilmiştir
- Firestore rule ihlalleri test edilmiştir

---

## ⚠️ Sorumluluk Reddi

Bu uygulama **eğitim ve prototip amaçlı** geliştirilmiştir.
Gerçek bir üretim ortamında ek olarak:
- Penetrasyon testleri
- Rate limiting
- Audit log sistemleri
önerilmektedir.

---

## 📌 Sonuç

Silencia, kullanıcı gizliliğini ve veri güvenliğini önceliklendiren,
modern güvenlik standartlarına uygun bir mimari ile geliştirilmiştir.

Uygulama; **kimlik doğrulama, veri izolasyonu ve servis ayrımı**
konularında güçlü ve sürdürülebilir bir yapı sunar.
