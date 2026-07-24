# Zero Trust Dönüşüm Serisi — GitHub Pages Sitesi

Bu klasör, Zero Trust Dönüşüm Serisi'ni **Just the Docs** temalı bir Jekyll sitesi olarak GitHub Pages üzerinde yayınlamaya hazır haldedir. Sol menüde Faz 0–4 altında gruplanmış bölümler, arama kutusu ve gezinme yapısı otomatik olarak gelir.

## Kurulum Adımları

### 1. GitHub'da yeni bir repo oluşturun

- Repo adı: `zero-trust-serisi` (proje sitesi olduğu için bu klasör adıyla eşleşmesi önerilir, farklı bir isim de seçebilirsiniz).
- Public (herkese açık) olmalı — GitHub Pages ücretsiz kullanım için public repo gerektirir (GitHub Pro/Team/Enterprise'da private repo'larda da Pages mümkündür).

### 2. `_config.yml` dosyasını güncelleyin

Bu klasördeki `_config.yml` dosyasını açın ve şu iki satırı kendi bilgilerinizle değiştirin:

```yaml
url: "https://KULLANICI_ADINIZ.github.io"
baseurl: "/zero-trust-serisi"   # repo adınızı farklı seçtiyseniz burayı da güncelleyin
```

### 3. Dosyaları GitHub'a push edin

Bu klasörün içeriğini (tüm dosyalar ve `.github/` klasörü dahil) yeni reponuzun kök dizinine push edin:

```bash
cd zero-trust-site
git init
git add .
git commit -m "Zero Trust dönüşüm serisi - ilk yayın"
git branch -M main
git remote add origin https://github.com/KULLANICI_ADINIZ/zero-trust-serisi.git
git push -u origin main
```

### 4. GitHub Pages'i etkinleştirin

1. Repo sayfasında **Settings → Pages** yolunu izleyin.
2. **Build and deployment → Source** kısmında **GitHub Actions**'ı seçin (Jekyll build/deploy dosyası zaten `.github/workflows/pages.yml` içinde hazır).
3. `main` branch'e her push yaptığınızda site otomatik olarak derlenip yayınlanacaktır.
4. Birkaç dakika içinde siteniz şu adreste yayında olacaktır:
   `https://KULLANICI_ADINIZ.github.io/zero-trust-serisi/`

### 5. (Opsiyonel) Yerelde önizleme

Yayınlamadan önce bilgisayarınızda test etmek isterseniz (Ruby kurulu olmalı):

```bash
bundle install
bundle exec jekyll serve
```

Ardından tarayıcıda `http://localhost:4000/zero-trust-serisi/` adresini açabilirsiniz.

## Klasör Yapısı

```
zero-trust-site/
├── _config.yml                  # Site ayarları (başlık, tema, url)
├── Gemfile                      # Ruby bağımlılıkları (yerel önizleme için)
├── .github/workflows/pages.yml  # Otomatik build & deploy
├── index.md                     # Ana sayfa
├── 00-giris-zero-trust-nedir.md # Giriş bölümü
├── faz0-index.md ... faz4-index.md  # Her fazın sol menüdeki üst kategori sayfası
└── 01-....md ... 14-....md      # 15 bölümün tamamı, front matter eklenmiş halde
```

## Yeni Bölüm Eklemek İsterseniz

Yeni bir `.md` dosyası oluşturup en üstüne şu şablonu ekleyin:

```yaml
---
layout: default
title: "Bölüm Başlığı (Sidebar'da Görünecek Kısa Ad)"
parent: "Faz X: Kategori Adı"
nav_order: N
---
```

`parent` alanı, hangi faz index sayfasının (`faz0-index.md` vb.) altına gruplanacağını belirler; `nav_order` ise o kategori içindeki sırayı belirler.

## Notlar

- Tema, GitHub'ın resmi olarak desteklediği `jekyll-remote-theme` eklentisiyle uzaktan (`remote_theme: just-the-docs/just-the-docs`) çekilir; bu nedenle GitHub Actions tabanlı deploy kullanılmalıdır (Settings → Pages → Source → **GitHub Actions**), "Deploy from a branch" seçeneği bu temayı derlemez.
- Arama kutusu, sayfa içi bağlantılar ve karanlık/açık tema geçişi Just the Docs tarafından otomatik sağlanır; ek bir yapılandırma gerekmez.
