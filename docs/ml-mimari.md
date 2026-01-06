---

## 🔐 Kod Gizliliği ve Repository Yapısı

Silencia uygulamasının makine öğrenmesi bileşeni, **özel veri setleri** ve
**eğitilmiş model dosyaları** içerdiği için **private repository** olarak
tutulmaktadır.

Bu nedenle:

- ML modelinin **eğitim kodları**
- Kullanılan **ham veri setleri**
- Model ağırlıkları (`.pkl`, `.h5`, vb.)
- Eğitim ve test pipeline’ları

**public olarak paylaşılmamaktadır.**

Bu repoda (showcase):

- ML mimarisinin **konsept tasarımı**
- Kullanılan teknolojiler
- Sistem akışı ve entegrasyon yaklaşımı

dokümantasyon seviyesinde sunulmaktadır.

---

## 🧠 Kullanılan Teknolojiler

- **Python**
- **MediaPipe** – El ve parmak landmark tespiti
- **OpenCV** – Görüntü ön işleme
- **Scikit-learn** – Random Forest sınıflandırıcı
- **NumPy / Pandas** – Veri işleme
- **REST API** – Mobil uygulama entegrasyonu

---

## ⚙️ Model Detayları

- **Model Türü:** Random Forest Classifier
- **Girdi (Input):**  
  - MediaPipe tarafından tespit edilen el landmark koordinatları  
  - Normalize edilmiş x, y, z değerleri
- **Çıktı (Output):**  
  - Tahmin edilen Türk İşaret Dili harfi
- **Çalışma Modu:**  
  - Gerçek zamanlı (frame bazlı tahmin)

---

## 📱 Mobil Uygulama Entegrasyonu

- Mobil uygulama (Flutter), kamera görüntüsünü alır
- Görüntü backend üzerindeki ML servisine gönderilir
- ML servisi:
  - Landmark tespiti yapar
  - Özellik çıkarımı uygular
  - Model üzerinden tahmin üretir
- Tahmin sonucu JSON formatında mobil uygulamaya döndürülür

---

## 🚀 Geliştirilebilirlik ve Ölçeklenebilirlik

- ML servisi **bağımsız** bir yapıdadır
- Model güncellendiğinde mobil uygulamada değişiklik gerekmez
- Yeni işaretler eklenerek model yeniden eğitilebilir
- Farklı modeller (CNN, LSTM vb.) ile değiştirilebilir yapıdadır

---

## 📌 Not

Bu doküman, Silencia projesinin makine öğrenmesi altyapısını
**tanıtım ve değerlendirme amacıyla** hazırlanmıştır.

Kaynak kodlara erişim, proje sahibinin iznine tabidir.
