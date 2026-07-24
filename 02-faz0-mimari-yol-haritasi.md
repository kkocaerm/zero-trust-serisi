---
layout: default
title: "Zero Trust Mimarisi ve Yol Haritası Tasarımı"
parent: "Faz 0: Hazırlık ve Değerlendirme"
nav_order: 2
---

# Bölüm 2 (Faz 0): Zero Trust Mimarisi ve Yol Haritası Tasarımı

## Yönetici Özeti

Olgunluk değerlendirmesi "neredeyiz" sorusunu cevaplar; bu bölüm "nereye ve hangi sırayla gidiyoruz" sorusunu cevaplar. Burada amaç, soyut Zero Trust ilkelerini kurumun mevcut Microsoft lisanslama yatırımı, NGFW/NAC altyapısı ve iş önceliklerine uyarlanmış somut bir referans mimariye ve fazlandırılmış uygulama planına dönüştürmektir. Bu plan olmadan yapılan ürün alımları, birbirinden bağımsız çalışan "nokta çözümler" yığınına dönüşür ve Zero Trust'ın temel vaadi olan entegre, sürekli risk değerlendirmesi gerçekleşmez.

## Referans Mimari: PEP, PDP, PA

NIST SP 800-207, Zero Trust mimarisini üç temel bileşenle tanımlar:

- **Policy Decision Point (PDP):** Erişim talebinin onaylanıp onaylanmayacağına karar veren motor. Kurumumuzda bu rol büyük ölçüde Microsoft Entra ID Conditional Access motoru, NGFW politika motorları (FortiGate/Panorama) ve Cisco ISE politika sunucusu tarafından paylaşılacaktır.
- **Policy Enforcement Point (PEP):** Kararın uygulandığı nokta — NGFW, NAC switch portu, ZTNA gateway, CASB proxy.
- **Policy Administrator (PA):** PDP'nin kararını PEP'e ileten bileşen.

Bu üçlüyü kurumsal ekosistemimize haritalarsak: Entra ID kimlik sinyallerini üretir ve karar verir; Intune/Defender for Endpoint cihaz sinyalini besler; Cisco ISE ve NGFW'ler (Fortinet/Palo Alto) ağ seviyesinde kararı uygular; CASB ve Purview uygulama/veri seviyesinde ek kontrol katmanı sağlar; Sentinel tüm sinyalleri merkezi görünürlük için toplar.

## Mimari Prensipler

Yol haritası tasarımı şu dört prensibe sadık kalmalıdır:

**Açık doğrulama (explicit verification):** Hiçbir erişim, sadece ağ konumuna (örn. "VPN üzerinden geliyor") dayanarak onaylanmaz; kimlik, cihaz durumu ve davranış birlikte değerlendirilir.

**En az ayrıcalık:** Just-in-time ve just-enough-access modeli; kalıcı yönetici rolleri yerine zaman sınırlı, onay gerektiren yükseltmeler.

**İhlal varsayımı (assume breach):** Mimari, bir bileşenin ele geçirildiği senaryoda yayılmayı sınırlayacak şekilde tasarlanır — bu, mikro-segmentasyonun (Faz 2) temel gerekçesidir.

**Sürekli izleme:** Erişim kararı tek seferlik değil, oturum boyunca yeniden değerlendirilir (örn. risk skoru oturum ortasında yükselirse erişim kesilir).

## Fazlandırma Mantığı ve Bağımlılıklar

Yol haritasının fazlara bölünmesi rastgele değildir; her faz bir öncekinin ürettiği sinyale dayanır:

Faz 1 (Kimlik) önce gelir, çünkü kimlik sinyali olmadan ne cihaz uyumluluğu ne de ağ segmentasyonu anlamlı bir karar motoruna bağlanabilir. Faz 2 (Cihaz ve Ağ), kimlik sinyaliyle birleşince "sadece uyumlu cihazdan, doğrulanmış kullanıcı erişebilir" gibi kararlar mümkün hale gelir. Faz 3 (Uygulama ve Veri), artık güvenilir kimlik ve cihaz sinyali üzerine, hangi verinin kim tarafından, hangi bağlamda görülebileceğini ekler. Faz 4 (Optimal), tüm bu sinyalleri tek bir sürekli risk motorunda (SASE + UEBA) birleştirir.

## Yol Haritası ve Zaman Çizelgesi Önerisi

Orta ölçekli bir kurum için tipik bir zaman çizelgesi:

- **0-3. ay (Faz 0):** Değerlendirme, mimari onay, hızlı kazanımlar (MFA zorunluluğu, eski protokollerin kapatılması).
- **3-9. ay (Faz 1):** Conditional Access, PIM, kimlik yaşam döngüsü.
- **9-18. ay (Faz 2):** Intune/Defender for Endpoint dağıtımı, Cisco ISE NAC devreye alma, NGFW politika yeniden tasarımı.
- **18-27. ay (Faz 3):** CASB/Defender for Cloud Apps, Purview etiketleme ve DLP, Sentinel entegrasyonu.
- **27-36. ay (Faz 4):** SASE birleşimi, sürekli uyarlanabilir güven motoru, optimizasyon döngüsü.

Bu zaman çizelgesi kurum büyüklüğüne, mevcut lisanslamaya (E3/E5) ve değişim yönetimi kapasitesine göre kısalıp uzayabilir; önemli olan bağımlılık sırasının korunmasıdır.

## KPI ve Yönetişim

Her faz için ölçülebilir KPI'lar tanımlanmalıdır: MFA kapsama %, koşullu erişim politikası kapsama %, yönetilen cihaz oranı, segmentasyon kuralı sayısı, DLP olay yanıt süresi gibi. Bir Zero Trust yönlendirme komitesi (BT, güvenlik, uyum, iş birimi temsilcileri) aylık olarak bu KPI'ları gözden geçirmelidir.

## Faz 0 Tamamlanma Kriterleri

- Referans mimari ve bileşen haritası üst yönetim tarafından onaylanmış olmalı.
- Fazlandırılmış yol haritası ve KPI seti belirlenmiş olmalı.
- Yönlendirme komitesi kurulmuş ve ilk bütçe onayı alınmış olmalı.

## Sonraki Bölüm

Bölüm 2.5: **Microsoft Policy Ekosistemi** — bu mimari prensiplerin ve PDP/PEP/PA modelinin, Microsoft ürünlerinde somut "policy" nesnelerine nasıl çevrildiğini ele alacağız.
