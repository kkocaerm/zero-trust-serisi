---
layout: default
title: "Cisco ISE ile Kimlik Tabanlı NAC"
parent: "Faz 2: Cihaz ve Ağ Güvenliği"
nav_order: 2
---

# Bölüm 6 (Faz 2): Ağ Erişim Kontrolü — Cisco ISE ile Kimlik Tabanlı NAC ve Mikro-segmentasyon

## Yönetici Özeti

Kimlik ve cihaz sinyalleri kurulduktan sonra, bu sinyallerin fiziksel/kablosuz ağ erişimine de yansıtılması gerekir. Geleneksel ağlarda bir cihaz prize takıldığında veya Wi-Fi'a bağlandığında geniş bir VLAN'a erişim kazanır; bu, Zero Trust'ın "ihlal varsayımı" prensibiyle doğrudan çelişir. Cisco ISE (Identity Services Engine), ağ erişimini kimlik ve cihaz duruşuna göre dinamik olarak karara bağlayan ve mikro-segmentasyonu mümkün kılan Network Access Control (NAC) platformudur.

Yönetim için bu yatırımın gerekçesi açıktır: bir fidye yazılımı veya saldırganın ele geçirdiği bir cihazın ağda yatay olarak yayılma kabiliyetini, NAC ve segmentasyon olmadan sınırlamak mümkün değildir.

## 802.1X Kimlik Doğrulama ve Profiling

Cisco ISE, ağa bağlanan her cihazı iki temel yöntemle tanır:

**802.1X (dot1x):** Cihaz veya kullanıcı, switch portuna/Wi-Fi'a bağlanırken dijital sertifika veya kullanıcı adı/şifre ile kimlik doğrular (EAP-TLS önerilir, çünkü sertifika tabanlıdır ve kimlik avına dayanıklıdır). Bu kimlik doğrulama, mümkünse Entra ID/AD ile entegre edilir, böylece ağ erişim kararı da kurumsal kimlikle uyumlu olur.

**Profiling (MAC Authentication Bypass + cihaz parmak izi):** 802.1X destekleyemeyen cihazlar (yazıcı, IP telefon, IoT sensörü) için ISE, DHCP, HTTP user-agent, SNMP gibi sinyalleri analiz ederek cihaz türünü otomatik olarak sınıflandırır ve uygun, sınırlı bir erişim profiline atar.

## TrustSec ve Security Group Tags (SGT) ile Mikro-segmentasyon

Cisco ISE'nin Zero Trust'a en güçlü katkısı TrustSec mimarisidir. Klasik VLAN tabanlı segmentasyon, IP adresine dayalıdır ve ağ büyüdükçe yönetimi karmaşıklaşır. TrustSec ise her cihaza/kullanıcıya, IP'den bağımsız bir **Security Group Tag (SGT)** atar (örn. "Finans_Kullanıcı", "IoT_Kamera", "Misafir"). Switch'ler ve güvenlik duvarları, bu etiketler arasında hangi iletişimin serbest olduğunu tanımlayan bir matris (SGT-to-SGT politika matrisi) ile çalışır.

Bu yaklaşımın avantajı: bir kullanıcı/cihaz ağda fiziksel olarak yer değiştirse (farklı switch'e, farklı binaya taşınsa) bile SGT etiketi ve buna bağlı erişim hakları değişmez. Bu, statik VLAN/ACL yönetiminin operasyonel yükünü ortadan kaldırır ve "her segment kendi içinde varsayılan olarak izole" mantığını (mikro-segmentasyon) mümkün kılar.

## Guest ve BYOD Onboarding

ISE, misafir ve BYOD cihazları için kendi kendine hizmet (self-service) bir onboarding portalı sunar: sponsor onayı, sertifika tabanlı cihaz kaydı (BYOD için), zaman sınırlı misafir erişimi. Bu cihazlar otomatik olarak kısıtlı bir SGT'ye (örn. "Internet_Only") atanır ve kurumsal kaynaklara hiçbir koşulda erişemez.

## pxGrid: Ekosistem Entegrasyonu

Cisco ISE'nin Zero Trust mimarisindeki en kritik rolü, **pxGrid** protokolü üzerinden diğer güvenlik araçlarıyla çift yönlü sinyal paylaşımıdır:

- Microsoft Defender for Endpoint veya Sentinel'den gelen bir tehdit tespiti, pxGrid üzerinden ISE'ye iletilebilir ve ISE bu cihazı otomatik olarak karantina SGT'sine taşıyabilir (Adaptive Network Control — ANC).
- Fortinet veya Palo Alto NGFW'ler, kullanıcı/grup bilgisini ISE'den pxGrid ile çekerek güvenlik duvarı kurallarını kimlik bazlı yazabilir.

Bu entegrasyon, Zero Trust'ın "sinyaller arası otomatik tepki" prensibinin ağ katmanındaki en somut örneğidir: bir uç noktada tespit edilen tehdit, dakikalar içinde ağ erişiminin otomatik kesilmesine dönüşür.

## Kablosuz Ağ ve IoT Özel Durumu

Kurumsal Wi-Fi'da WPA2/WPA3-Enterprise ile 802.1X zorunlu kılınmalı, IoT cihazları ise kendi izole SGT/VLAN'larına (kamera, HVAC, akıllı sensörler) ayrılmalı ve bu cihazların yalnızca ihtiyaç duydukları belirli hedeflerle (örn. üretici bulut servisi) iletişim kurmasına izin verilmelidir — varsayılan olarak diğer her şeye erişimleri reddedilir.

## Faz 2 (Adım 2) Tamamlanma Kriterleri

- Kablolu ve kablosuz ağda 802.1X (mümkünse EAP-TLS) zorunlu hale getirilmiş olmalı.
- SGT tabanlı segmentasyon matrisi tanımlanmış ve kritik segmentler (finans, IoT, misafir) ayrıştırılmış olmalı.
- pxGrid entegrasyonu ile en az bir otomatik karantina senaryosu (EDR tetiklemeli) test edilmiş olmalı.
- Guest/BYOD onboarding süreci self-service hale gelmiş olmalı.

## Sonraki Bölüm

Bölüm 7: **NGFW ile Zero Trust Network Access — Fortinet FortiGate Mimarisi** — ağ segmentasyonunu, internet/WAN çıkışı ve uygulama seviyesi kontrolle nasıl tamamlayacağımızı ele alacağız.
