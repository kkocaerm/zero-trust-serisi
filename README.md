# Zero Trust Dönüşüm Serisi — GitHub Pages Sitesi

Bu klasör, Zero Trust Dönüşüm Serisi'ni **Just the Docs** temalı bir Jekyll sitesi olarak GitHub Pages üzerinde yayınlamaya hazır haldedir. Sol menüde Faz 0–4 altında gruplanmış bölümler, arama kutusu ve gezinme yapısı otomatik olarak gelir.

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
