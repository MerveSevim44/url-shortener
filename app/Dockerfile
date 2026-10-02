# 1. ADIM: Temel imajı seç
# Python 3.11'in hafif (slim) versiyonunu kullanıyoruz
# "slim" = gereksiz araçlar yok, imaj boyutu küçük
FROM python:3.11-slim

# 2. ADIM: Container içinde çalışma klasörünü belirle
# Uygulama dosyaları buraya kopyalanacak
WORKDIR /app

# 3. ADIM: Önce SADECE bağımlılık dosyasını kopyala
# (Neden önce? Docker katman önbellekleme sayesinde,
# kod değişse bile bağımlılıkları tekrar yüklemez → hızlı build)
COPY requirements.txt .

# 4. ADIM: Python bağımlılıklarını yükle
RUN pip install --no-cache-dir -r requirements.txt

# 5. ADIM: Geri kalan tüm uygulama dosyalarını kopyala
COPY . .

# 6. ADIM: Uygulamanın hangi portu kullanacağını belirt
# (Bu sadece dokümantasyon amaçlı, portu açmaz)
EXPOSE 5000

# 7. ADIM: Container başladığında çalışacak komut
CMD ["python", "app.py"]
