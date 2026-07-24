---
layout: default
title: "SASE Mimarisine Geçiş"
parent: "Faz 4: Optimal Olgunluk"
nav_order: 1
---

# Bölüm 12 (Faz 4): SASE Mimarisine Geçiş

## Yönetici Özeti

Faz 1-3'te kurulan mimari, kimlik, cihaz, ağ, uygulama ve veri katmanlarında güçlü ama hâlâ kısmen ayrık kontrol noktalarından oluşur: şube/veri merkezindeki NGFW, uzaktan erişim için ayrı bir ZTNA/VPN çözümü, bulut uygulamaları için ayrı bir CASB. Secure Access Service Edge (SASE), Gartner'ın tanımladığı şekliyle, ağ fonksiyonlarını (SD-WAN) ve güvenlik fonksiyonlarını (NGFW, SWG, CASB, ZTNA) tek bir bulut-yerel platformda, tek bir politika motoruyla birleştiren mimaridir.

Yönetim için SASE'nin gerekçesi hem güvenlik hem maliyet/operasyon yönlüdür: kullanıcı nerede olursa olsun (ofis, ev, seyahat) aynı politika tutarlılığıyla korunur ve trafiğin şirket veri merkezine geri yönlendirilmesi (backhaul) ihtiyacı ortadan kalkar, bu da gecikmeyi azaltır ve altyapı maliyetini düşürür.

## SASE'nin Bileşenleri: SD-WAN + SSE

SASE iki ana bileşenin birleşimidir:

**SD-WAN:** Şubeler/veri merkezi arası bağlantıyı, geleneksel MPLS hatları yerine internet üzerinden, akıllı yönlendirme ve performans optimizasyonuyla sağlar.

**Security Service Edge (SSE):** SWG (Secure Web Gateway), CASB, ZTNA ve FWaaS (Firewall as a Service) fonksiyonlarını bulut üzerinden, kullanıcının bulunduğu her yere en yakın noktadan (Point of Presence — PoP) sunan güvenlik katmanı.

Bir kurum SD-WAN'ı olmadan da SSE'yi (yalnızca güvenlik tarafını) benimseyebilir — bu, mevcut WAN altyapısını değiştirmeden Zero Trust güvenlik kontrollerini buluta taşımak isteyen kurumlar için pratik bir ilk adımdır.

## Fortinet ve Palo Alto'nun SASE Teklifleri

Bu seri boyunca ele aldığımız iki NGFW üreticisi de kendi SASE platformlarını sunar:

**FortiSASE:** Fortinet'in bulut tabanlı SSE/SASE teklifi; FortiGate'in Security Fabric mimarisini (Bölüm 7) buluta genişletir, FortiClient ZTNA ajanını aynı agent olarak kullanır ve FortiAnalyzer ile aynı merkezi loglama/analitiğe entegre olur. Mevcut FortiGate altyapısı olan kurumlar için en az sürtünmeli geçiş yolu budur.

**Prisma SASE (Prisma Access + Prisma SD-WAN):** Palo Alto'nun teklifi, GlobalProtect/Prisma Access (Bölüm 8) üzerine inşa edilir ve aynı App-ID/User-ID politika motorunu bulut PoP'larına taşır; Panorama ile merkezi politika yönetimi sürdürülür.

Her iki üreticinin de stratejisi aynıdır: veri merkezi/şubede kurulan NGFW politika dilini, değiştirmeden bulut PoP'larına genişletmek — bu, Faz 2'de yapılan yatırımın Faz 4'te tekrar kullanılmasını (re-use) sağlar.

## Microsoft Entra Internet Access ve Private Access (Global Secure Access)

Microsoft'un SSE teklifi, Entra ID ile derinlemesine entegre olan Global Secure Access şemsiyesi altındaki iki üründür:

**Entra Internet Access:** Genel internet ve SaaS trafiği için, Conditional Access politikalarının doğrudan uygulandığı bir güvenli web gateway'i; kimlik bazlı erişim kararını, kullanıcı internete her çıktığında uygular.

**Entra Private Access:** Geleneksel VPN'in yerini alan, uygulama bazlı ZTNA çözümü; FortiClient ZTNA veya Prisma Access ile kavramsal olarak aynı amacı taşır, ancak doğrudan Entra ID Conditional Access motoruyla bütünleşiktir (yani aynı risk skoru, aynı cihaz uyumluluk sinyali, hem uygulama erişiminde hem internet erişiminde tutarlı şekilde kullanılır).

## Karma (Hibrit) SASE Stratejisi: Hangi Üretici Nerede?

Çoğu kurum tek bir SASE üreticisine tam geçiş yapmadan önce, mevcut yatırımlarını koruyarak karma bir strateji izler:

- Mevcut Fortinet/Palo Alto NGFW yatırımı olan kurum, şube/veri merkezi güvenliğini aynı üreticinin SASE teklifiyle genişletir (FortiSASE veya Prisma SASE).
- Microsoft E5 lisansına sahip kurum, kimlikle en derin entegrasyonu istediği internet/SaaS erişim senaryoları için Entra Internet/Private Access'i tamamlayıcı olarak devreye alır.
- Bu iki katman, Sentinel'de (Bölüm 11) ortak görünürlük altında birleştirilir.

Önemli olan, hangi üretici seçilirse seçilsin, politika motorunun **tek bir kaynaktan** (idealde kimlik sağlayıcısı) beslenmesi ve tüm erişim noktalarında (şube, ev, mobil, bulut) aynı tutarlılıkla uygulanmasıdır.

## Faz 4 (Adım 1) Tamamlanma Kriterleri

- En az bir SSE bileşeni (FortiSASE, Prisma Access veya Entra Internet/Private Access) tüm kullanıcılar için aktif olmalı.
- Şube/veri merkezi NGFW politikaları ile bulut SASE politikaları arasında tutarlılık sağlanmış olmalı.
- Geleneksel full-tunnel VPN kullanımı, ölçülebilir şekilde azalmış olmalı.
- SASE platformunun logları Sentinel'e entegre edilmiş olmalı.

## Sonraki Bölüm

Bölüm 13: **Sürekli Uyarlanabilir Güven — UEBA, AI/ML Tabanlı Tehdit Tespiti ve Otomatik Müdahale** — Optimal seviyenin tanımlayıcı özelliği olan, sürekli ve dinamik risk değerlendirmesini ele alacağız.
