---
layout: default
title: "Microsoft Sentinel ile SIEM/SOAR"
parent: "Faz 3: Uygulama ve Veri Güvenliği"
nav_order: 3
---

# Bölüm 11 (Faz 3): Görünürlük ve Otomasyon — Microsoft Sentinel ile SIEM/SOAR Entegrasyonu

## Yönetici Özeti

Bu seride Faz 1-3 boyunca Entra ID, Intune/Defender for Endpoint, Cisco ISE, Fortinet/Palo Alto NGFW, CASB ve Purview olmak üzere altı farklı sinyal kaynağı kurduk. Bu sinyaller kendi başlarına değerlidir, ancak Zero Trust'ın vaat ettiği gerçek güç, bu sinyallerin **tek bir merkezi platformda birleştirilip ilişkilendirilmesinden** gelir. Microsoft Sentinel, bulut-yerel bir SIEM (Security Information and Event Management) ve SOAR (Security Orchestration, Automation and Response) platformu olarak bu birleştirme rolünü üstlenir.

Yönetim için bu fazın gerekçesi, ortalama tespit ve müdahale süresinin (MTTD/MTTR) doğrudan iş riskine ve potansiyel ihlal maliyetine etkisidir; parçalı araçlarla çalışan bir güvenlik ekibi, saatler/günler süren manuel korelasyon yapmak zorunda kalır, merkezi bir SIEM/SOAR ise bu süreci dakikalara indirir.

## Veri Konektörleri: Tüm Ekosistemin Tek Noktada Toplanması

Sentinel'in gücü, geniş bir konektör ekosisteminden gelir:

- **Microsoft 365 Defender / Entra ID:** Kimlik oturum açma logları, risk tespitleri, cihaz uyumluluk olayları, e-posta tehdit tespitleri.
- **Cisco ISE:** pxGrid veya syslog üzerinden ağ erişim olayları, karantina aksiyonları.
- **Fortinet FortiGate / Palo Alto Networks:** Syslog/CEF formatında güvenlik duvarı, IPS ve uygulama kontrol logları.
- **Defender for Cloud Apps:** CASB tarafından tespit edilen anormal bulut uygulaması davranışları.
- **Purview:** DLP olayları, hassasiyet etiketi uygulama/ihlal kayıtları.

Tüm bu kaynaklar, ortak bir şema (Advanced Security Information Model — ASIM) üzerinden normalize edilerek, üretici/ürün bağımsız sorgular ve korelasyon kuralları yazılabilir hale gelir.

## Analytics Rules ve Korelasyon Senaryoları

Sentinel'in asıl katma değeri, tek bir kaynaktaki anormalliği değil, **kaynaklar arası örtüşen sinyalleri** tespit etmesidir. Örnek bir korelasyon senaryosu: "Entra ID'de bir kullanıcının oturum riski 'yüksek' olarak işaretlendi VE aynı kullanıcı son 1 saat içinde Defender for Cloud Apps'te anormal hacimde dosya indirdi VE ISE loglarında bu kullanıcının cihazı son 24 saatte yeni bir MAC adresinden bağlandı." Bu üç sinyal tek başına düşük öncelikli olabilir, ancak birlikte değerlendirildiğinde yüksek öncelikli bir olay (incident) olarak otomatik açılır.

## UEBA: Kullanıcı ve Varlık Davranış Analitiği

Sentinel'in UEBA (User and Entity Behavior Analytics) motoru, her kullanıcı ve cihaz için bir davranışsal taban çizgisi (baseline) oluşturur ve bu taban çizgisinden sapmaları (alışılmadık oturum açma saati, alışılmadık erişilen kaynak, alışılmadık veri hacmi) puanlar. Bu, statik kurallarla yakalanamayan, "düşük ve yavaş" (low-and-slow) saldırı tekniklerini tespit etmede kritik rol oynar ve Faz 4'te ele alınacak "sürekli uyarlanabilir güven" modelinin temel veri girdisidir.

## SOAR: Playbook Tabanlı Otomatik Yanıt

Sentinel'in Logic Apps tabanlı playbook motoru, tespit edilen olaylara otomatik yanıt tanımlamayı sağlar. Tipik playbook örnekleri:

- Yüksek riskli oturum tespit edildiğinde, kullanıcının Entra ID oturumunu otomatik olarak sonlandır ve MFA'yı zorla yeniden doğrulat.
- Bir uç noktada fidye yazılımı davranışı tespit edildiğinde, Defender for Endpoint üzerinden cihazı otomatik izole et VE pxGrid üzerinden Cisco ISE'ye bildirim gönderip cihazı karantina SGT'sine taşı.
- CASB'de anormal toplu indirme tespit edildiğinde, ilgili kullanıcının erişim token'larını iptal et ve güvenlik ekibine Teams/e-posta bildirimi gönder.

Bu otomasyon, Zero Trust'ın "sürekli değerlendirme" prensibini, insan müdahalesi beklemeden saniyeler/dakikalar içinde uygulamaya döker.

## Threat Hunting: Proaktif Arama

Otomatik kurallara ek olarak, Sentinel'in Kusto Query Language (KQL) tabanlı threat hunting arayüzü, güvenlik analistlerinin henüz bir kural tarafından yakalanmamış şüpheli desenleri proaktif olarak araması imkânını verir. Olgun bir Zero Trust programında, threat hunting bulguları düzenli olarak yeni analytics rule'lara dönüştürülerek tespit kapasitesi sürekli genişletilir.

## Üçüncü Taraf NGFW/NAC Verisinin Değeri

Burada vurgulanması gereken kritik nokta: Sentinel'in değeri sadece Microsoft ürünlerinden gelen verilerle sınırlı değildir. Fortinet ve Palo Alto loglarının, Cisco ISE olaylarının aynı platforma akması, "tek bir üreticinin görmediği resmi" tamamlar — örneğin bir NGFW'nin engellediği bir C2 (command-and-control) bağlantı denemesi ile aynı zaman aralığında bir uç noktada görülen şüpheli süreç, ayrı ayrı görüldüğünde önemsiz, birlikte görüldüğünde kritik bir göstergedir.

## Faz 3 (Adım 3) Tamamlanma Kriterleri

- Tüm ana sinyal kaynakları (Entra ID, Defender ailesi, Cisco ISE, NGFW'ler, CASB, Purview) Sentinel'e bağlanmış olmalı.
- En az 5-10 kaynaklar arası korelasyon kuralı (analytics rule) devrede olmalı.
- En az 3-5 yüksek öncelikli senaryo için otomatik SOAR playbook'u tanımlanmış olmalı.
- Düzenli (örn. aylık) threat hunting çalışması operasyonel hale gelmiş olmalı.

## Sonraki Bölüm

Bölüm 12: **SASE Mimarisine Geçiş** — Faz 4'e geçişle birlikte, ağ ve güvenlik fonksiyonlarını bulut tabanlı, tek bir politika motorunda nasıl birleştireceğimizi ele alacağız.
