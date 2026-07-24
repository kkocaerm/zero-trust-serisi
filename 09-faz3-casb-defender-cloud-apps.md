---
layout: default
title: "CASB ve Microsoft Defender for Cloud Apps"
parent: "Faz 3: Uygulama ve Veri Güvenliği"
nav_order: 1
---

# Bölüm 9 (Faz 3): Bulut Uygulama Güvenliği — CASB ve Microsoft Defender for Cloud Apps

## Yönetici Özeti

Faz 2'nin sonunda kimlik, cihaz ve ağ katmanları kontrol altına alınmıştır. Ancak modern kurumlarda veri artık şirket veri merkezinde değil, onlarca/yüzlerce SaaS uygulamasında (Microsoft 365, Salesforce, Workday, kişisel Dropbox/Google Drive hesapları dahil) yaşamaktadır. Cloud Access Security Broker (CASB), kullanıcı ile bulut uygulaması arasına yerleşerek bu görünürlük ve kontrol boşluğunu kapatan katmandır.

Yönetim için bu yatırımın gerekçesi, "Shadow IT" riskidir: BT departmanının bilgisi/onayı olmadan kullanılan bulut uygulamaları (genellikle çalışanların kişisel hesaplarıyla kurumsal veri yüklediği servisler), kurumun en büyük ama en az görünen veri sızıntısı risklerinden biridir. Bölüm 2.5'teki Microsoft Policy tablosunda bu fazın karşılığı, Defender for Cloud Apps'in Session, Access ve File Policy nesneleridir.

## CASB'nin Dört Sütunu

Sektör standardı olarak CASB çözümleri dört temel işlevi yerine getirir:

**Görünürlük (Visibility):** Kurum genelinde hangi bulut uygulamalarının kullanıldığının (onaylı ve onaysız) keşfi — genellikle güvenlik duvarı/proxy loglarının analiziyle (Shadow IT Discovery) yapılır.

**Uyumluluk (Compliance):** Kullanılan uygulamaların düzenleyici gerekliliklere (KVKK, GDPR, sektörel standartlar) uygunluğunun risk skorlamasıyla değerlendirilmesi.

**Veri Güvenliği (Data Security):** Bulut uygulamalarına yüklenen/paylaşılan içerikte hassas veri (kimlik numarası, kredi kartı, gizli belge etiketi) tespiti ve DLP uygulanması.

**Tehdit Koruması (Threat Protection):** Anormal indirme/paylaşım davranışı, ele geçirilmiş hesap belirtileri, kötü amaçlı dosya paylaşımının tespiti.

## Microsoft Defender for Cloud Apps Mimarisi

Microsoft'un CASB çözümü olan Defender for Cloud Apps, üç farklı entegrasyon yöntemini bir arada kullanır:

**Cloud Discovery (Log Collector / NGFW Entegrasyonu):** Fortinet ve Palo Alto NGFW'lerin ürettiği trafik logları, Defender for Cloud Apps'e aktarılarak kurum genelinde kullanılan tüm bulut uygulamalarının (onaylı/onaysız) bir envanteri ve risk skoru çıkarılır. Bu, Faz 2'de kurulan NGFW altyapısının Faz 3'teki en doğrudan getirisidir.

**API Connector (Sanctioned Apps):** Microsoft 365, Salesforce, Box gibi onaylı/entegre uygulamaların API'leri üzerinden geçmiş veriler taranır, anormal paylaşım/indirme davranışları tespit edilir ve otomatik düzeltme (örn. aşırı paylaşılan dosyanın paylaşımını kaldırma) uygulanabilir.

**Conditional Access App Control (Reverse Proxy):** Entra ID Conditional Access ile entegre çalışan bu mod, kullanıcı oturumunu gerçek zamanlı bir ters proxy üzerinden geçirerek **oturum içinde** kontrol uygular — örneğin yönetilemeyen bir cihazdan gelen kullanıcının hassas bir dosyayı indirmesini engelleyebilir, ancak görüntülemesine izin verebilir.

## Session Policy Örneği: Gerçek Zamanlı Kontrol

Tipik bir oturum politikası şu şekilde çalışır: "Yönetilemeyen (uyumsuz) bir cihazdan SharePoint Online'a erişen kullanıcı, 'Gizli' etiketli belgeleri görüntüleyebilir ama indiremez veya yazdıramaz." Bu politika, Bölüm 5'te kurulan cihaz uyumluluk sinyalini, Bölüm 10'da ele alınacak Purview hassasiyet etiketleriyle birleştirerek, hem kimlik hem cihaz hem veri sınıflandırmasını tek bir kararda harmanlar — bu, Zero Trust'ın "Entegre" olgunluk seviyesinin somut bir göstergesidir.

## Üçüncü Taraf CASB ve NGFW Entegrasyonu

Kurumun mevcut Fortinet/Palo Alto altyapısı, Defender for Cloud Apps'in (veya başka bir CASB ürününün) görünürlük katmanını besler; tersi yönde de NGFW'ler, CASB'nin tespit ettiği yüksek riskli/onaysız uygulamaları otomatik olarak engelleme kuralına dönüştürebilir (Fabric Connector veya benzer entegrasyon API'leri ile). Bu döngü, CASB'nin "tespit eden", NGFW'nin "uygulayan" bileşen olduğu tamamlayıcı bir mimari oluşturur.

## Shadow IT'den Sanctioned App'e Geçiş Stratejisi

CASB keşfinin tipik sonucu, kurumda yüzlerce onaysız bulut uygulamasının kullanıldığının görülmesidir. Bu noktada pragmatik yaklaşım, tüm uygulamaları tek seferde engellemek değil, risk skoruna göre kademeli bir strateji izlemektir: yüksek riskli/düşük kullanımlı uygulamalar engellenir, orta riskli ama yaygın kullanılanlar için kurumsal alternatif sağlanır (örn. kişisel Dropbox yerine OneDrive), düşük riskli uygulamalar izlemeye alınır.

## Faz 3 (Adım 1) Tamamlanma Kriterleri

- NGFW logları ile Cloud Discovery entegrasyonu kurulmuş ve Shadow IT envanteri çıkarılmış olmalı.
- En az 3-5 kritik SaaS uygulaması için API connector aktif olmalı.
- Yönetilemeyen cihaz senaryosu için en az bir oturum politikası (indirme/yazdırma kısıtlaması) devrede olmalı.
- Yüksek riskli onaysız uygulamalar için engelleme/yönlendirme stratejisi uygulanmış olmalı.

## Sonraki Bölüm

Bölüm 10: **Veri Sınıflandırma ve Koruma — Microsoft Purview ile Bilgi Koruması ve DLP** — verinin kendisini, nerede olursa olsun nasıl koruyacağımızı ele alacağız.
