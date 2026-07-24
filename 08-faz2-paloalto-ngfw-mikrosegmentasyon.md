---
layout: default
title: "Palo Alto Networks ile Mikro-segmentasyon"
parent: "Faz 2: Cihaz ve Ağ Güvenliği"
nav_order: 4
---

# Bölüm 8 (Faz 2): NGFW ile Mikro-segmentasyon — Palo Alto Networks App-ID ve Prisma Access

## Yönetici Özeti

Palo Alto Networks, Zero Trust kavramının pazarlama dilinde yaygınlaşmasında öncü rol oynamış bir üreticidir ve kendi Zero Trust metodolojisini (5 adımlı yaklaşım) ürün portföyüyle doğrudan eşleştirir. Bu bölüm, kurumların Fortinet yerine veya Fortinet ile birlikte (örn. veri merkezinde Palo Alto, şube/uzaktan erişimde Fortinet gibi karma senaryolarda) Palo Alto NGFW ve Prisma Access ile aynı Zero Trust hedeflerine nasıl ulaşabileceğini ele alır. Çoğu kurum tek bir NGFW üreticisi seçer; bu bölüm Fortinet'e alternatif/karşılaştırmalı bir teknik derinlik sağlamak amacıyla yazılmıştır.

## App-ID ve User-ID: Kimlik ve Uygulama Farkındalığı

Palo Alto'nun NGFW mimarisinin temel yapı taşları:

**App-ID:** Trafiği port/protokole değil, uygulama imzasına göre sınıflandırır; bir uygulamanın port değiştirerek (örn. 443 üzerinden tünelleme) kuralları atlatmasını önler.

**User-ID:** Güvenlik duvarı kurallarını IP adresi yerine kullanıcı/grup kimliğine bağlar. Entra ID/AD ile entegre edilerek "Finans grubu, sadece finans uygulamasına HTTPS üzerinden erişebilir" gibi kimlik bazlı politikalar yazılabilir. Bu, Cisco ISE'den gelen SGT bilgisiyle de eşleştirilerek tutarlı bir politika dili kurulabilir.

**Content-ID:** Tehdit imzaları, URL filtreleme ve veri sızıntısı imzalarını tek bir tarama motorunda birleştirir.

## Palo Alto'nun 5 Adımlı Zero Trust Metodolojisi

Palo Alto, Zero Trust uygulamasını şu sıralı adımlarla tanımlar ve bu seri boyunca izlediğimiz fazlandırmayla büyük ölçüde örtüşür:

1. **Koruma yüzeyini tanımla:** Tüm ağı korumaya çalışmak yerine, en kritik veri/varlık/uygulama/servisi (DAAS — Data, Applications, Assets, Services) önceliklendir.
2. **İşlem akışlarını haritalama:** Bu kritik varlıklara kimin, nasıl, hangi protokolle eriştiğini çıkar.
3. **Zero Trust mimarisini tasarla:** Mikro-segment sınırlarını (segmentasyon ağ geçitleri) bu akışlara göre yerleştir.
4. **Zero Trust politikasını oluştur:** Kimi-Ne-Ne zaman-Nereye-Neden-Nasıl (Kipling metodolojisi) sorularına cevap veren ayrıntılı, en az ayrıcalıklı kurallar yaz.
5. **İzle ve sürdür:** Trafiği günlükle, analiz et, politikaları sürekli iyileştir.

Bu metodoloji, Bölüm 2'de tanımladığımız genel mimari prensiplerin NGFW seviyesinde nasıl uygulamaya döküleceğini gösterir.

## Mikro-segmentasyon: Zone-Based ve Host-Based Yaklaşım

Palo Alto, mikro-segmentasyonu iki tamamlayıcı seviyede uygular:

**Ağ tabanlı (zone-based):** Veri merkezinde farklı güven seviyelerine sahip segmentler (DMZ, uygulama katmanı, veritabanı katmanı) arasına NGFW yerleştirilir; her geçiş App-ID/User-ID ile denetlenir.

**İş yükü tabanlı (host-based, CN-Series/VM-Series ile):** Konteyner ve sanal makine seviyesinde, aynı segment içindeki iş yükleri arası (east-west) trafiği de denetleyerek "bir kez içeri girilince her şeye erişim" riskini ortadan kaldırır — bu, bulut-yerel (cloud-native) iş yüklerinin arttığı kurumlar için kritik bir yetenektir.

## GlobalProtect ve Prisma Access: Uzaktan ve Bulut Erişimi

**GlobalProtect**, geleneksel istemci tabanlı VPN ajanıdır; cihaz duruş kontrolü (HIP — Host Information Profile) yaparak cihazın güvenlik durumuna göre erişim seviyesini ayarlar.

**Prisma Access**, Palo Alto'nun bulut tabanlı Security Service Edge (SSE) teklifidir ve şirket veri merkezine yönlendirme yapmadan, kullanıcının bulunduğu her yerden en yakın bulut güvenlik noktasına (PoP) bağlanarak NGFW yeteneklerini (App-ID, tehdit önleme, URL filtreleme) ve ZTNA'yı bulut üzerinden sunar. Bu, Bölüm 12'de ele alınacak SASE mimarisinin Palo Alto tarafındaki temel bileşenidir.

## Panorama: Merkezi Politika Yönetimi

Çok sayıda fiziksel/sanal Palo Alto güvenlik duvarının (veri merkezi, şube, bulut) tutarlı politika ile yönetilmesi Panorama üzerinden sağlanır. Bu, özellikle büyük ve dağıtık kurumlarda politika tutarsızlığını (her şubenin kendi kuralını yazması riskini) önler ve merkezi denetim/loglama sağlar.

## Faz 2 (Adım 4) Tamamlanma Kriterleri

- User-ID entegrasyonu ile en azından kritik uygulamalarda kimlik bazlı kurallar yazılmış olmalı.
- DAAS önceliklendirmesi yapılmış ve en kritik 3-5 varlık için mikro-segment sınırları çizilmiş olmalı.
- Uzaktan erişimde GlobalProtect HIP kontrolü aktif olmalı.
- Çoklu cihaz ortamında Panorama ile merkezi politika yönetimi kurulmuş olmalı.

## Sonraki Bölüm

Bölüm 9: **Bulut Uygulama Güvenliği — CASB ve Microsoft Defender for Cloud Apps** — Faz 3 ile birlikte kontrolü ağ katmanından uygulama ve veri katmanına taşıyacağız.
