---
layout: default
title: "Mevcut Durum Analizi ve Olgunluk Değerlendirmesi"
parent: "Faz 0: Hazırlık ve Değerlendirme"
nav_order: 1
---

# Bölüm 1 (Faz 0): Mevcut Durum Analizi ve Zero Trust Olgunluk Değerlendirmesi

## Yönetici Özeti

Zero Trust dönüşümü, ürün satın alımıyla başlayan bir proje değil, mevcut durumun dürüst bir şekilde fotoğraflanmasıyla başlayan bir programdır. Mimariyi veya ürün yığınını seçmeden önce kurumun kimlik, cihaz, ağ, uygulama ve veri alanlarında hangi olgunluk seviyesinde olduğunu bilmeden atılacak her adım, ya gereksiz yatırım ya da kör nokta riski doğurur. Bu bölüm, dönüşüme başlamadan önce yapılması gereken envanter çıkarma, olgunluk skorlama ve risk önceliklendirme çalışmasını ele alır.

Yönetim kurulu ve üst yönetim için bu fazın çıktısı, "neredeyiz, nereye gitmemiz gerekiyor ve bu yolculuk hangi sırayla, ne kadar bütçeyle yapılmalı" sorularına somut cevap veren bir olgunluk raporu ve yol haritası taslağıdır.

## Envanter Çıkarma: Beş Sütun Üzerinden

Değerlendirme, CISA ZTMM'nin beş sütununa paralel ilerler:

**Kimlik:** Kaç farklı kimlik deposu var (on-prem AD, Entra ID, SaaS yerel kimlikler)? MFA kapsama oranı nedir? Ayrıcalıklı hesapların sayısı ve kalıcı (standing) yönetici erişimi var mı?

**Cihazlar:** Şirket envanterinde kaç cihaz var, kaçı yönetilebilir (MDM/MAM kapsamında)? BYOD politikası mevcut mu? Uç nokta koruması (AV/EDR) kapsama oranı nedir?

**Ağ:** Ağ düz mü (flat) yoksa VLAN/segment bazlı mı? Mevcut NAC çözümü var mı? Güvenlik duvarı kuralları ne sıklıkla denetleniyor, "any-any" kural sayısı nedir?

**Uygulama ve İş Yükleri:** Kritik uygulamalar VPN üzerinden mi erişiliyor, yoksa uygulama bazlı erişim var mı? Bulut iş yükleri (IaaS/PaaS/SaaS) için merkezi bir görünürlük var mı?

**Veri:** Veri sınıflandırma politikası var mı, varsa fiilen uygulanıyor mu? DLP kontrolleri hangi kanallarda (e-posta, depolama, uç nokta) aktif?

## Olgunluk Skorlama Yöntemi

Her sütun için 1-5 arası bir olgunluk skoru kullanılması önerilir:

1. **Yok / Ad-hoc:** Kontrol tanımlı değil veya tutarsız uygulanıyor.
2. **Tanımlı:** Politika var, ancak manuel ve parçalı uygulanıyor.
3. **Yönetilen:** Merkezi araçlarla uygulanıyor, ancak alanlar arası entegrasyon yok.
4. **Entegre:** Sinyaller alanlar arası paylaşılıyor (örn. cihaz uyumluluğu kimlik kararını etkiliyor).
5. **Optimal:** Gerçek zamanlı, otomatik, sürekli risk bazlı karar verme.

Bu skorlama, hem teknik ekiplerle atölye çalışmaları hem de mevcut araçların (Entra ID, mevcut NGFW, NAC, SIEM) loglarının analiziyle desteklenmelidir; sadece anket bazlı öz-değerlendirme yanıltıcı sonuçlar verebilir.

## Risk Önceliklendirme

Olgunluk skorları çıktıktan sonra, her boşluk (gap) şu kriterlere göre önceliklendirilir:

- **Saldırı yüzeyi etkisi:** Bu boşluk kapatılmazsa olası bir ihlalin yayılma alanı ne kadar büyür?
- **Uygulama maliyeti ve süresi:** Mevcut lisanslarla (örn. Microsoft E5) mı kapatılabilir, yeni yatırım mı gerekir?
- **İş etkisi:** Kullanıcı deneyimine ve operasyonel sürekliliğe etkisi nedir?

Tipik olarak en yüksek öncelik, MFA kapsama oranındaki boşluklar ve ayrıcalıklı hesaplardaki kalıcı erişim olur; çünkü bunlar düşük maliyetle kapatılabilir ve saldırı yüzeyini orantısız şekilde küçültür.

## Çıktı: Olgunluk Raporu ve Paydaş Haritası

Bu fazın somut çıktıları şunlardır: beş sütun için olgunluk skor kartı, önceliklendirilmiş boşluk listesi, paydaş haritası (BT, güvenlik, uygulama sahipleri, hukuk/uyum, iş birimleri) ve üst yönetime sunulacak bir özet sunum. Bu rapor, Bölüm 2'de detaylandırılacak mimari ve yol haritası tasarımının girdisidir.

## Faz 0 Tamamlanma Kriterleri

- Beş sütunun tamamı için olgunluk skoru belirlenmiş olmalı.
- En az ilk 90 günde ele alınacak 5-10 öncelikli boşluk listelenmiş olmalı.
- Üst yönetim, mevcut durumu ve risk gerekçesini onaylamış olmalı.

## Sonraki Bölüm

Bölüm 2: **Zero Trust Mimarisi ve Yol Haritası Tasarımı** — bu değerlendirmenin çıktılarını referans mimariye ve fazlandırılmış bir uygulama planına dönüştüreceğiz.
