---
layout: default
title: "Koşullu Erişim ve Risk Tabanlı Kimlik Doğrulama"
parent: "Faz 1: Kimlik Temeli"
nav_order: 2
---

# Bölüm 4 (Faz 1): Koşullu Erişim Politikaları ve Risk Tabanlı Kimlik Doğrulama

## Yönetici Özeti

Bölüm 3'te kurulan kimlik temeli üzerine, bu bölümde "her erişim talebine aynı kuralı uygulamak" yerine "bağlama göre farklılaştırılmış kararlar vermek" mantığına geçiyoruz. Bu, Zero Trust'ın "Gelişmiş" olgunluk seviyesine geçişin kalbidir: kullanıcı kim, hangi cihazdan, nereden, hangi uygulamaya ve ne risk seviyesiyle erişmek istiyor — bu sinyallerin tümü birlikte değerlendirilir.

Yönetim açısından bu yaklaşımın değeri, güvenliği artırırken kullanıcı deneyimini de iyileştirmesidir: düşük riskli, bilinen bir bağlamdan gelen erişim sürtünmesiz geçer, yüksek riskli erişim ise ek doğrulama ister veya tamamen engellenir.

## Conditional Access Motorunun Yapısı

Microsoft Entra Conditional Access, "sinyal → karar → uygulama" mantığıyla çalışır:

**Sinyaller (atamalar):** Kullanıcı/grup, bulut uygulaması, cihaz platformu, konum (IP/coğrafya), oturum riski, kullanıcı riski, kimlik doğrulama gücü.

**Kararlar (erişim kontrolleri):** Erişimi engelle, MFA iste, cihazın uyumlu olmasını iste (Intune entegrasyonu), parola değişikliği zorunlu kıl, oturum süresini sınırla.

**Oturum kontrolleri:** Koşullu erişim uygulama kontrolü (CASB ile entegre — Bölüm 9'da detaylandırılacak), indirme/yazdırma kısıtlaması, sürekli erişim değerlendirmesi (Continuous Access Evaluation — CAE).

Örnek bir politika seti şu şekilde olabilir: "Finans uygulamasına erişim, yalnızca uyumlu (Intune kayıtlı) cihazdan ve kurum ağı dışından geliyorsa ek MFA gerektirir; yüksek riskli oturumlarda erişim tamamen engellenir."

## Entra ID Protection: Risk Sinyalleri

Entra ID Protection (P2 lisansı), makine öğrenmesiyle iki tür risk skoru üretir:

- **Kullanıcı riski:** Sızdırılmış kimlik bilgileri, alışılmadık davranış geçmişi gibi sinyallerle hesabın uzun vadeli risk profili.
- **Oturum riski:** Anonim IP, imkânsız seyahat (impossible travel), yeni cihaz/konum gibi anlık oturum sinyalleri.

Bu risk skorları Conditional Access politikalarına doğrudan girdi olarak bağlanır; örneğin "orta riskli oturumda MFA iste, yüksek riskli oturumda parola sıfırlamasını zorunlu kıl" gibi otomatik kurallar tanımlanabilir. Bu, Zero Trust'ın "sürekli değerlendirme" prensibinin pratikte nasıl çalıştığının en somut örneğidir.

## Privileged Identity Management (PIM): Just-in-Time Erişim

En az ayrıcalık prensibinin en güçlü uygulaması, kalıcı yönetici rollerinin ortadan kaldırılmasıdır. PIM ile:

- Yönetici rolleri varsayılan olarak **pasif** atanır; kullanıcı role ihtiyaç duyduğunda **aktivasyon** talep eder.
- Aktivasyon, onay süreci, MFA tekrar doğrulaması ve gerekçe girilmesi gerektirebilir.
- Roller otomatik olarak zaman sınırlıdır (örn. 4-8 saat) ve süre sonunda otomatik geri alınır.
- Tüm aktivasyonlar denetim (audit) günlüğüne kaydedilir ve Sentinel'e beslenebilir.

Bu yaklaşım, ele geçirilmiş bir yönetici hesabının saldırgan için kalıcı bir "altın anahtar" olmasını önler.

## Access Reviews ve Entitlement Management

Zaman içinde biriken "yetki şişmesi" (permission creep) sorununu çözmek için periyodik erişim incelemeleri (Access Reviews) ve kendi kendine hizmet erişim talep paketleri (Entitlement Management) kullanılır. Bu, özellikle gruplara ve uygulamalara üyeliklerin düzenli olarak gözden geçirilip onaylanmasını/iptal edilmesini otomatikleştirir.

## Örnek Politika Mimarisi (Olgunluk Bazlı)

Gelişmiş seviyede tipik bir politika katmanlaması:

1. Tüm kullanıcılar → temel MFA.
2. Ayrıcalıklı roller → PIM + ek kimlik doğrulama gücü (FIDO2) + onay süreci.
3. Yüksek hassasiyetli uygulamalar → uyumlu cihaz + bilinen konum + oturum riski düşük olmalı.
4. Konuk/harici kullanıcılar → daha sıkı oturum kontrolleri ve sınırlı oturum süresi.

## Faz 1 Tamamlanma Kriterleri

- Conditional Access politikaları, en azından kimlik + cihaz + risk sinyallerini birleştirecek şekilde devreye alınmış olmalı.
- Tüm ayrıcalıklı roller PIM ile just-in-time modele geçirilmiş olmalı.
- Periyodik erişim incelemeleri operasyonel hale gelmiş olmalı.
- Kullanıcı ve oturum risk skorları, otomatik politika tetikleyicisi olarak kullanılıyor olmalı.

## Sonraki Bölüm

Bölüm 5: **Uç Nokta Güvenliği: Microsoft Intune + Defender for Endpoint** — Faz 2'ye geçişle birlikte, cihaz sinyalinin kimlik kararlarına nasıl entegre edileceğini ele alacağız.
