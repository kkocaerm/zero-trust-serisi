---
layout: default
title: "Fortinet FortiGate ile NGFW / ZTNA"
parent: "Faz 2: Cihaz ve Ağ Güvenliği"
nav_order: 3
---

# Bölüm 7 (Faz 2): NGFW ile Zero Trust Network Access — Fortinet FortiGate Mimarisi

## Yönetici Özeti

Cisco ISE, erişim katmanında (kenar switch/Wi-Fi) kimlik bazlı kontrolü sağlarken, yeni nesil güvenlik duvarları (NGFW) bu kontrolü ağ segmentleri arası geçişlerde, internet çıkışında ve veri merkezi/bulut sınırlarında uygular. Fortinet FortiGate, Security Fabric mimarisiyle bu kontrolü tek bir politika ve görünürlük çatısı altında birleştirmeyi hedefler. Bu bölüm, FortiGate'in Zero Trust mimarisindeki rolünü ve ZTNA (Zero Trust Network Access) yeteneklerini ele alır.

İş değeri açısından Fortinet yatırımı, hem geleneksel güvenlik duvarı/IPS/web filtreleme fonksiyonlarını hem de uzaktan erişim ve segmentasyon görevlerini tek bir konsolide platformda birleştirerek operasyonel karmaşıklığı azaltır.

## NGFW Temel Yetenekleri ve Zero Trust'a Katkısı

Klasik bir güvenlik duvarı port/protokol bazlı kural çalıştırırken, FortiGate gibi bir NGFW şu ek katmanları sağlar:

- **Application Control:** Trafiği port numarasına göre değil, gerçek uygulama imzasına göre tanır (örn. 443 portundaki trafiğin Dropbox mu, kurumsal SaaS mı olduğunu ayırt eder).
- **SSL/TLS Inspection:** Şifreli trafiğin içeriğini denetleyerek (uygun sertifika yönetimiyle) şifreli kanal içinde gizlenen tehditleri yakalar — modern trafiğin büyük çoğunluğu şifreli olduğundan bu kritik bir kontroldür.
- **IPS (Intrusion Prevention System):** Bilinen açık imzalarına ve davranışsal anomalilere göre saldırı girişimlerini engeller.
- **Identity-based Policy:** Kural tabanını IP adresi yerine kullanıcı/grup kimliğine bağlama (FSSO — Fortinet Single Sign-On veya doğrudan Entra ID/LDAP entegrasyonu ile).

Bu yeteneklerin Zero Trust'a katkısı, "ağ konumuna güvenme" yerine "trafiğin kim/ne tarafından, hangi uygulama için üretildiğini bilerek karar verme" mantığına geçilmesidir.

## FortiClient ZTNA: Uygulama Bazlı Erişim

Geleneksel VPN, kullanıcıya tüm ağa (veya geniş bir alt ağa) erişim açar — bu, Zero Trust'ın doğrudan zıttıdır. FortiClient ZTNA agent, bunun yerine her uygulama erişimini ayrı ayrı doğrular:

1. Kullanıcı FortiClient üzerinden belirli bir uygulamaya (örn. dahili bir web uygulaması) erişim talep eder.
2. FortiClient, cihaz duruş kontrolü yapar (işletim sistemi yaması, AV durumu, sertifika varlığı) ve bu bilgiyi bir **ZTNA tag** olarak üretir.
3. FortiGate, bu tag'i ve kullanıcı kimliğini birlikte değerlendirerek erişimi sadece talep edilen uygulamaya, port seviyesinde açar — ağın geri kalanı görünmez kalır.

Bu model, VPN'in "tünele girince her şeye erişim" sorununu çözer ve uzaktan çalışan kullanıcılar için de mikro-segmentasyon prensibini uygular.

## Fortinet Security Fabric: Entegre Görünürlük

Security Fabric, FortiGate, FortiClient (uç nokta), FortiAnalyzer (log/analitik) ve FortiManager (merkezi politika yönetimi) bileşenlerini tek bir mimaride birleştirir. Bu, çok şubeli/dağıtık kurumlarda tutarlı politika uygulamasını ve merkezi görünürlüğü sağlar. Security Fabric, ayrıca üçüncü taraf entegrasyonları (Fabric Connector) üzerinden Cisco ISE'den kullanıcı/SGT bilgisi çekebilir veya Microsoft Sentinel'e log gönderebilir — bu, Faz 3'te ele alınacak merkezi görünürlük katmanının temelini oluşturur.

## SD-WAN ve Şube Segmentasyonu

Çok şubeli kurumlarda FortiGate'in SD-WAN yetenekleri, şubeler arası trafiği güvenlik politikasından bağımsız bırakmaz; her şube kendi içinde segmentlenir (misafir Wi-Fi, IoT, kurumsal kullanıcı ayrımı) ve şubeler arası iletişim de internet çıkışındaki kadar sıkı denetlenir — "iç ağ güvenlidir" varsayımı şube seviyesinde de geçerli değildir.

## Veri Merkezi / Bulut Segmentasyonu

Veri merkezi içinde FortiGate VM veya FortiGate-VM bulut varyantları, iş yükleri arası (east-west) trafiği denetleyebilir; bu, bir web sunucusunun ele geçirilmesi durumunda saldırganın doğrudan veritabanı sunucusuna sıçramasını önleyen kritik bir mikro-segmentasyon katmanıdır.

## Faz 2 (Adım 3) Tamamlanma Kriterleri

- Kimlik bazlı güvenlik duvarı politikaları (FSSO/Entra ID entegrasyonu) devreye alınmış olmalı.
- SSL inspection, kritik trafik kategorilerinde aktif olmalı.
- Uzaktan erişimde geleneksel full-tunnel VPN, ZTNA uygulama bazlı erişimle kademeli olarak değiştirilmiş olmalı.
- Veri merkezi/bulut iş yükleri arasında en az temel east-west segmentasyon uygulanmış olmalı.

## Sonraki Bölüm

Bölüm 8: **NGFW ile Mikro-segmentasyon — Palo Alto Networks App-ID ve Prisma Access** — benzer hedefleri Palo Alto Networks ekosisteminde nasıl gerçekleştireceğimizi ele alacağız.
