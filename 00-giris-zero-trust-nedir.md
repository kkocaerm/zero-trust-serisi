---
layout: default
title: "Giriş: Zero Trust Nedir?"
nav_order: 2
---

# Zero Trust Dönüşüm Serisi — Giriş: Zero Trust Nedir, Neden Şimdi?

## Yönetici Özeti

Geleneksel ağ güvenliği modeli, "kurumsal ağın içi güvenlidir, dışı tehlikelidir" varsayımına dayanır. Hibrit çalışma, bulut uygulamaları, üçüncü taraf entegrasyonları ve fidye yazılımı saldırılarının yatay yayılma (lateral movement) becerisi bu varsayımı geçersiz kılmıştır. Bir saldırganın tek bir kullanıcı kimlik bilgisini veya tek bir cihazı ele geçirmesi artık tüm kurumsal ağa erişim anlamına gelebiliyor.

Zero Trust (Sıfır Güven), NIST SP 800-207 standardında tanımlandığı şekliyle, "hiçbir kullanıcıya, cihaza veya iş yüküne, ağdaki konumuna bakılmaksızın varsayılan olarak güvenilmemesi" ilkesine dayanan bir güvenlik mimarisidir. Her erişim talebi; kimlik, cihaz durumu, konum, davranış ve istenen kaynağın hassasiyeti gibi sinyallere göre dinamik olarak değerlendirilir ve en az ayrıcalık ilkesiyle yetkilendirilir.

Bu seri, bir kurumun klasik çevre güvenliğinden (perimeter security) başlayıp, Microsoft güvenlik ekosistemi (Entra ID, Intune, Defender ailesi, Purview, Sentinel), NGFW çözümleri (Fortinet, Palo Alto Networks), ağ erişim kontrolü (Cisco ISE), CASB ve SASE mimarilerini kullanarak en olgun Zero Trust seviyesine nasıl ulaşabileceğini, faz faz ve hem yönetici hem teknik bakış açısıyla ele alacaktır.

## Zero Trust'ın Beş Sütunu

CISA Zero Trust Maturity Model (ZTMM) ve Microsoft'un kendi olgunluk modeli, Zero Trust'ı beş temel alan üzerinden yapılandırır:

- **Kimlik (Identity):** Kullanıcı ve servis hesaplarının güçlü, sürekli doğrulanan kimlikleri.
- **Cihazlar (Devices):** Erişim talep eden her uç noktanın sağlık ve uyumluluk durumu.
- **Ağ (Network):** Düz (flat) ağ yerine mikro-segmentasyon ve şifreli iletişim.
- **Uygulama ve İş Yükleri (Applications & Workloads):** Uygulama seviyesinde erişim kontrolü ve güvenli geliştirme.
- **Veri (Data):** Verinin sınıflandırılması, etiketlenmesi ve nerede olursa olsun korunması.

Bu beş sütunu birbirine bağlayan çapraz yetkinlikler ise görünürlük & analitik, otomasyon & orkestrasyon ve yönetişimdir (governance).

## Olgunluk Modeli: Geleneksel → Gelişmiş → Optimal

Microsoft ve CISA'nın olgunluk modelleri üç (bazı versiyonlarda dört) seviye tanımlar:

1. **Geleneksel (Traditional / Faz 0-1):** Statik politikalar, ağ konumuna dayalı güven, manuel envanter ve uyumluluk.
2. **Gelişmiş (Advanced / Faz 2-3):** Sinyaller arası entegrasyon başlar; kimlik, cihaz ve ağ kararları birbirini besler; mikro-segmentasyon ve otomatik politika uygulama devreye girer.
3. **Optimal (Faz 4):** Tüm sinyaller gerçek zamanlı, sürekli ve otomatik olarak değerlendirilir; erişim kararları statik kurallar değil, dinamik risk skorlarıyla verilir; self-healing ve AI destekli tehdit avcılığı standart hale gelir.

## Bu Serinin Yol Haritası

| Faz | Odak | Bu Seride İşlenen Ürünler |
|---|---|---|
| Faz 0 | Değerlendirme ve Mimari Tasarım | Olgunluk değerlendirmesi, referans mimari |
| Faz 1 | Kimlik Temeli | Microsoft Entra ID, Conditional Access, PIM |
| Faz 2 | Cihaz ve Ağ | Intune, Defender for Endpoint, Cisco ISE, Fortinet NGFW, Palo Alto NGFW |
| Faz 3 | Uygulama ve Veri | CASB / Defender for Cloud Apps, Microsoft Purview, Microsoft Sentinel |
| Faz 4 | Optimal Seviye | SASE (FortiSASE, Prisma SASE, Entra Internet/Private Access), sürekli uyarlanabilir güven |

Her bölümde önce ilgili fazın **iş değeri ve risk gerekçesi** (yönetici özeti), ardından **mimari ve uygulama detayları** (teknik derinlik) ele alınacak ve bölüm sonunda "bir sonraki faza geçiş kriterleri" verilecektir. Bu yapı, hem CISO seviyesinde yatırım gerekçesi sunmaya hem de mühendislik ekiplerinin uygulama planı çıkarmasına imkân tanır.

## Sonraki Bölüm

Bölüm 1: **Mevcut Durum Analizi ve Zero Trust Olgunluk Değerlendirmesi** — dönüşüme başlamadan önce kurumun nerede olduğunu nasıl ölçeceğimizi ele alacağız.
