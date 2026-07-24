---
layout: default
title: "Microsoft Intune + Defender for Endpoint"
parent: "Faz 2: Cihaz ve Ağ Güvenliği"
nav_order: 1
---

# Bölüm 5 (Faz 2): Uç Nokta Güvenliği — Microsoft Intune ve Defender for Endpoint

## Yönetici Özeti

Kimlik temeli kurulduktan sonra Zero Trust'ın bir sonraki sütunu cihazdır. Çalınmış bir kimlik bilgisi, eğer talep eden cihaz uyumsuz veya tehlikeye girmişse, erişim kararını değiştirmelidir. Bu bölüm, cihazların envantere alınması, uyumluluk durumunun sürekli izlenmesi ve bu durumun kimlik katmanındaki (Conditional Access) kararlara doğrudan girdi olarak nasıl bağlanacağını ele alır.

İş değeri açıdan bu faz, hem güvenlik açığını kapatır hem de BT operasyonlarını standartlaştırır: cihaz yapılandırması, yama yönetimi ve uyumluluk merkezi olarak yönetilir hale gelir. Bölüm 2.5'te tanımladığımız Microsoft Policy ekosisteminde cihaz sütününün somut karşılığı, burada ele alınan Intune Compliance Policy ve Configuration Profile nesneleridir.

## Microsoft Intune ile Cihaz Yönetimi

Intune, hem şirket cihazlarını (Mobile Device Management — MDM) hem de BYOD/kişisel cihazları (Mobile Application Management — MAM) yönetebilir:

**MDM (tam yönetim):** Cihaz kayıt altına alınır, disk şifreleme (BitLocker/FileVault), parola/PIN politikası, uygulama dağıtımı ve uzaktan silme (remote wipe) merkezi olarak uygulanır.

**MAM (uygulama bazlı yönetim):** Cihaz tam olarak yönetilmez, ancak kurumsal verinin bulunduğu uygulama (örn. Outlook, Teams) bir "uygulama koruma politikası" ile sarılır — kopyala/yapıştır kısıtlaması, uygulama PIN'i, kurumsal verinin kişisel uygulamalara sızmasını engelleme. BYOD senaryolarında Zero Trust'ı, cihazın tamamını yönetmeden uygulamak için kritik bir araçtır.

## Uyumluluk Politikaları ve Conditional Access Entegrasyonu

Intune'da tanımlanan uyumluluk politikaları (işletim sistemi sürümü, disk şifreleme açık mı, jailbreak/root tespiti, antivirüs durumu) cihaza bir "uyumlu/uyumsuz" etiketi verir. Bu etiket, Bölüm 4'te kurulan Conditional Access politikalarına doğrudan bağlanır: "kurumsal kaynaklara erişim, yalnızca uyumlu cihazdan yapılabilir" kuralı, kimlik ve cihaz sinyalini birleştiren ilk somut Zero Trust kontrolüdür.

## Microsoft Defender for Endpoint: EDR ve Ötesi

Defender for Endpoint (MDE), uç noktada şu yetenekleri sağlar:

- **Saldırı Yüzeyi Azaltma (Attack Surface Reduction):** Makro tabanlı kötü amaçlı yazılımları, fileless saldırı tekniklerini ve şüpheli script davranışlarını önceden tanımlanmış kurallarla engeller.
- **Endpoint Detection & Response (EDR):** Uç noktadaki süreç, dosya ve ağ davranışlarını sürekli izler; şüpheli zincirleri (örn. Office belgesinden PowerShell çalıştırma) tespit eder.
- **Otomatik Soruşturma ve Yanıt (AIR):** Tespit edilen olaylarda otomatik olarak kanıt toplama, etkilenen varlığı izole etme ve gerekirse otomatik düzeltme uygulama.
- **Cihaz Risk Skoru:** MDE'nin ürettiği cihaz risk skoru, Intune uyumluluk durumuna ek bir sinyal olarak Conditional Access'e beslenebilir (Microsoft Defender for Endpoint Conditional Access entegrasyonu) — yani cihaz "uyumlu" olsa bile aktif bir tehdit tespit edilirse erişim otomatik olarak kısıtlanabilir.

## Yama Yönetimi ve Yapılandırma Sertleştirme

Zero Trust'ın "ihlal varsayımı" prensibi, bilinen güvenlik açıklarının (CVE) hızlı kapatılmasını da kapsar. Intune üzerinden Windows Update for Business halkaları (ring) tanımlanarak yama dağıtımı kademeli ve otomatik hale getirilir. Ayrıca güvenlik temeli (security baseline) şablonları ile işletim sistemi yapılandırması (gereksiz servislerin kapatılması, yerel yönetici hesaplarının sınırlandırılması) standartlaştırılır.

## BYOD ve Yönetilemeyen Cihaz Senaryosu

Tüm cihazların MDM kapsamına alınamayacağı senaryolarda (yüklenici, kişisel cihaz, IoT) Zero Trust mimarisi şu katmanlı yaklaşımı önerir: MAM ile uygulama seviyesinde koruma, oturum tabanlı CASB kontrolü (indirme engelleme, salt-okunur erişim) ve ağ seviyesinde NAC ile profil bazlı sınırlama (Bölüm 6). Böylece yönetilemeyen cihazlar tamamen dışlanmadan, riskleri orantılı şekilde sınırlandırılır.

## Faz 2 (Adım 1) Tamamlanma Kriterleri

- Şirket cihazlarının büyük çoğunluğu Intune'da kayıtlı ve uyumluluk politikasına bağlı olmalı.
- Cihaz uyumluluğu, Conditional Access politikalarında zorunlu bir koşul haline gelmiş olmalı.
- Defender for Endpoint tüm uç noktalarda aktif ve otomatik soruşturma/yanıt modunda çalışıyor olmalı.
- BYOD için en az MAM tabanlı bir koruma politikası devrede olmalı.

## Sonraki Bölüm

Bölüm 6: **Ağ Erişim Kontrolü: Cisco ISE ile Kimlik Tabanlı NAC** — cihaz ve kimlik sinyalini, kablolu/kablosuz ağ erişim kararına nasıl taşıyacağımızı ele alacağız.
