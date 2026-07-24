---
layout: default
title: "Microsoft Policy Ekosistemi ve Policy as Code"
parent: "Faz 0: Hazırlık ve Değerlendirme"
nav_order: 3
---

# Bölüm 2.5 (Faz 0): Microsoft Policy Ekosistemi — Zero Trust Politikalarının Tanımlanması ve Uygulanması

## Yönetici Özeti

Bölüm 2'de Zero Trust mimarisini PDP (Policy Decision Point), PEP (Policy Enforcement Point) ve PA (Policy Administrator) bileşenleriyle tanımladık. Microsoft güvenlik ekosisteminde bu soyut mimari, somut ve tek bir ortak isimle karşımıza çıkar: **Policy (Politika)**. Entra ID, Intune, Defender ailesi, Purview, Defender for Cloud ve Sentinel — bu ürünlerin hepsi, güvenlik kararlarını "policy" adı verilen, tanımlanabilen, belirli bir kapsama atanabilen ve denetlenebilen nesneler üzerinden ifade eder.

Bu bölüm iki ayrı soruyu yanıtlar: *Politikalar nerede, nasıl tanımlanır?* ve *Tanımlanan politikalar nasıl değerlendirilir ve uygulanır?* Bu iki boyutu anlamadan kurulan kontroller ya tutarsız çalışır ya da yanlış sırayla birbirini geçersiz kılar; bu da Zero Trust programının güvenlik açıklarını kapatmak yerine operasyonel kaos yaratmasına yol açar.

---

## 1. Policy Ekosistemi: Ürün–Politika–Sütun Haritası

| Ürün | Policy Nesnesi | Nerede Oluşturulur | Zero Trust Sütunu |
|---|---|---|---|
| Microsoft Entra ID | Conditional Access Policy | entra.microsoft.com → Koruma → Koşullu Erişim | Kimlik |
| Microsoft Entra ID Governance | Access Review Policy, Entitlement Management Policy, Lifecycle Workflow | entra.microsoft.com → Kimlik Yönetimi | Kimlik |
| Microsoft Entra ID Protection | Risk Policy (User Risk / Sign-in Risk) | entra.microsoft.com → Koruma → Identity Protection | Kimlik |
| Microsoft Intune | Compliance Policy | intune.microsoft.com → Cihazlar → Uyumluluk | Cihaz |
| Microsoft Intune | Configuration Profile | intune.microsoft.com → Cihazlar → Yapılandırma | Cihaz |
| Microsoft Intune | Security Baseline | intune.microsoft.com → Uç Nokta Güvenliği → Güvenlik Temelleri | Cihaz |
| Microsoft Intune | App Protection Policy (MAM) | intune.microsoft.com → Uygulamalar → Uygulama Koruma İlkeleri | Cihaz |
| Microsoft Defender for Endpoint | Attack Surface Reduction (ASR) Policy | security.microsoft.com → Güvenlik Açıkları / Uç Nokta Güvenliği | Cihaz |
| Microsoft Defender for Endpoint | Antivirus / EDR Policy | security.microsoft.com → Uç Nokta Güvenliği → Virüsten Koruma | Cihaz |
| Microsoft Defender for Cloud Apps | Session Policy | security.microsoft.com → Bulut Uygulamaları → Politikalar | Uygulama (CASB) |
| Microsoft Defender for Cloud Apps | Access Policy | security.microsoft.com → Bulut Uygulamaları → Politikalar | Uygulama (CASB) |
| Microsoft Defender for Cloud Apps | File Policy | security.microsoft.com → Bulut Uygulamaları → Politikalar | Uygulama (CASB) |
| Microsoft Defender for Cloud Apps | App Discovery Policy | security.microsoft.com → Bulut Uygulamaları → Cloud Discovery | Uygulama (CASB) |
| Microsoft Purview | DLP Policy | compliance.microsoft.com → Veri Kaybı Önleme | Veri |
| Microsoft Purview | Sensitivity Label & Auto-labeling Policy | compliance.microsoft.com → Bilgi Koruması | Veri |
| Microsoft Purview | Retention Policy / Label | compliance.microsoft.com → Veri Yaşam Döngüsü | Veri |
| Microsoft Purview | Insider Risk Policy | compliance.microsoft.com → İçeriden Risk Yönetimi | Veri |
| Microsoft Defender for Cloud | Security Policy / Initiative | portal.azure.com → Defender for Cloud → Ortam Ayarları | İş Yükleri |
| Azure Policy | Policy / Initiative (Assignment) | portal.azure.com → Azure Policy | İş Yükleri / Altyapı |
| Microsoft Sentinel | Analytics Rule | portal.azure.com → Sentinel → Analiz | Çapraz (Görünürlük) |
| Microsoft Sentinel | Automation Rule | portal.azure.com → Sentinel → Otomasyon | Çapraz (Otomasyon) |
| Microsoft Purview Compliance Manager | Düzenleyici çerçeve eşlemeleri (KVKK, GDPR, ISO 27001) | compliance.microsoft.com → Uyumluluk Yöneticisi | Yönetişim |

---

## 2. Ortak Policy Yapısı: Koşul → Kapsam → Etki

Microsoft ürünleri arasındaki farklı adlandırmalara karşın, tüm policy nesneleri aynı üç bileşeni içerir:

**Kapsam (Scope / Assignment):** Politika kime veya neye uygulanacak? Kullanıcı grubu, cihaz grubu, Azure aboneliği, içerik konumu (SharePoint site, posta kutusu). Her politikada kapsam tanımı ilk adımdır; kapsamsız (veya tüm kiracıyı kapsayan) bir politika, test aşamasında ciddi operasyonel kesintilere yol açabilir.

**Koşullar (Conditions / Signals):** Politikanın tetikleneceği koşul kümesi. "Kullanıcı grubu X'te mi?", "Cihaz uyumlu mu?", "Oturum riski yüksek mi?", "Dosya içeriğinde kredi kartı deseni var mı?", "Azure kaynağı etiket gerektiriyor mu?" gibi sinyaller bu kısımda tanımlanır. Koşullar AND/OR mantığıyla birleştirilebilir.

**Etki (Effect / Action / Grant/Session Control):** Koşullar karşılandığında ne olacak? Her ürünün kendi etkiler sözlüğü vardır (aşağıda ürün bazında detaylandırılmaktadır), ancak genel olarak dört kategoride toplanır: *İzin ver*, *Engelle*, *Denetle*, *Düzelt/İyileştir*.

---

## 3. Ürün Bazında Politika Oluşturma Mekaniği

### 3.1 Entra ID — Conditional Access Policy

Conditional Access (CA), Microsoft'un en merkezi Zero Trust politika motorudur; diğer ürünlerin sinyallerini (Intune uyumluluğu, MDE cihaz riski, Entra ID Protection risk skoru) kendi koşul bloklarına dahil edebilir.

**Oluşturma adımları:**

1. entra.microsoft.com → Koruma → Koşullu Erişim → Yeni Politika.
2. **Atamalar (Assignments):** Kullanıcı ve gruplar seçilir; tüm kullanıcılara uygulanacaksa "Tüm kullanıcılar" seçilir ama mutlaka bir "Hariç tut" grubu (acil erişim hesapları — break glass) eklenir.
3. **Hedef kaynaklar:** Belirli bulut uygulamaları mı, tüm uygulamalar mı?
4. **Koşullar:** Oturum riski seviyesi, cihaz platformu, konum (adlandırılmış konumlar veya ülkeler), istemci uygulaması (modern auth mı, eski protokol mü?), cihaz filtresi (yönetilen mi, uyumlu mu?).
5. **Erişim kontrolleri — Verme:** Erişimi engelle veya şu koşullarla ver: MFA gerektir, uyumlu cihaz gerektir, Entra ID'ye hibrit katılmış cihaz gerektir, onaylı istemci uygulaması gerektir. Bu kontroller AND/OR ile birleştirilebilir.
6. **Erişim kontrolleri — Oturum:** Uygulama uygulanan kısıtlamalar (app-enforced), Koşullu Erişim Uygulama Kontrolü (Defender for Cloud Apps proxy), oturum süresi, kalıcı tarayıcı oturumu, sürekli erişim değerlendirmesi.
7. **Politika durumu:** "Yalnızca Rapor" (Report-only) modu — politika canlıya alınmadan, oturum açma loglarında etkisinin simüle edilmesi. **Bu en kritik güvenlik önlemidir; tüm yeni CA politikaları önce Report-only modda test edilmeli**, ardından pilot gruba (halka) uygulanmalı, son olarak tüm kapsama açılmalıdır.

**Değerlendirme sırası:** Bir kullanıcı için birden fazla CA politikası geçerliyse, hepsi aynı anda değerlendirilir ve en kısıtlayıcı sonuç uygulanır (en kısıtlayıcı politika kazanır — "Block" her zaman "Allow"u geçer).

**Named Locations (Adlandırılmış Konumlar):** IP aralıkları veya ülkeler gruplandırılarak "Şirket ağı" veya "İzin verilen ülkeler" gibi mantıksal konumlar tanımlanır. CA politikaları bu konumlara göre farklı davranış sergileyebilir — örneğin şirket ağından gelen isteklerde MFA atlanabilir, yabancı ülkelerden gelen istekler ise tamamen engellenir.

---

### 3.2 Microsoft Intune — Compliance Policy ve Configuration Profile

**Compliance Policy (Uyumluluk İlkesi):**

1. intune.microsoft.com → Cihazlar → Uyumluluk → İlke Oluştur → Platform seç (Windows, iOS/iPadOS, Android, macOS).
2. **Ayarlar:** Minimum işletim sistemi sürümü, BitLocker/FileVault şifreleme zorunlu mu, Secure Boot aktif mi, cihaz uyarısı yok (jailbreak/root değil), TPM chip varlığı, Defender for Endpoint'ten gelen cihaz tehdit seviyesi (Low/Medium/High/Clear).
3. **Uyumsuzluk eylemleri:** Hemen mi, 1 gün sonra mı, 7 gün sonra mı uyumsuz sayılsın? E-posta bildirim gönder, son kullanıcıyı kilitle, cihazı devre dışı bırak.
4. **Kapsam etiketleri (Scope Tags):** Çok bölgeli/çok departmanlı kurumlarda policy yönetimini yetkilendirmek için kullanılır; belirli IT ekiplerinin yalnızca kendi kapsamlarındaki politikaları görmesini/yönetmesini sağlar.

Uyumluluk politikası değerlendirmesi periyodiktir (genellikle 8 saatte bir) ancak bir cihaz Intune'a ilk kez bağlandığında ve politika güncellendiğinde tetiklenir. Sonuç ("Uyumlu" / "Uyumsuz" / "Değerlendirilmedi"), Entra ID'nin Conditional Access politikasına gerçek zamanlı olarak sinyal olarak akar.

**Configuration Profile (Yapılandırma Profili):**

Uyumluluk politikasından farklı olarak, Configuration Profile cihazın nasıl yapılandırılacağını tanımlar (zorunlu kılar, kısıtlar). Örnek: USB portunu devre dışı bırak, belirli bir Wi-Fi profilini zorunlu yükle, BitLocker'ı etkinleştir ve kurtarma anahtarını Intune'a yedekle, macOS cihazlarda güvenlik duvarını aç. Bu profiller ADMX şablonları (Windows için), Settings Catalog veya OMA-URI ile tanımlanabilir.

**Security Baseline (Güvenlik Temeli):**

Microsoft'un yayımladığı hazır en iyi uygulama profilleridir (Windows güvenlik temeli, Defender for Endpoint temeli, Edge temeli). Yüzlerce güvenlik ayarını tek bir tıklamayla gruplamayı sağlar; Zero Trust başlangıç noktası için idealdir, özelleştirmeler üstüne eklenir.

**App Protection Policy (MAM — Uygulama Koruma İlkesi):**

Cihazın kendisi Intune MDM kapsamında olmasa bile (BYOD, kişisel cihazlar), kurumsal verinin bulunduğu uygulamayı (Outlook, Teams, OneDrive) kapsayan politika nesnesidir. Tanımlar: minimum uygulama sürümü, uygulama PIN zorunluluğu, kopyala/yapıştır kısıtlaması, kurumsal verinin "yönetilmeyen" uygulamalara (kişisel not defteri, kişisel bulut) sızdırılmasını engelleme, cihaz rootlanmış/jailbroken ise erişimi kes.

---

### 3.3 Microsoft Defender for Endpoint — ASR ve Antivirüs Politikaları

**Attack Surface Reduction (ASR) Kuralları:**

security.microsoft.com → Uç Nokta Güvenliği → Saldırı Yüzeyi Azaltma → İlke Oluştur yoluyla veya Intune üzerinden tanımlanır. Her ASR kuralı üç moddan birinde çalışabilir: **Denetim** (yalnızca logla, engelleme), **Engelle** (eylemi engelle ve logla), **Devre Dışı**.

Kritik ASR kuralları arasında şunlar öne çıkar: Office uygulamalarının çocuk süreç oluşturmasını engelle (makro saldırıları), obfuscated script'lerin çalışmasını engelle (fileless saldırılar), PSExec ve WMI komutlarının süreç oluşturmasını engelle (lateral movement), USB/çıkarılabilir medyadan gelen güvenilmeyen süreçlerin çalışmasını engelle.

**Uygulama önerisi:** Her ASR kuralı, ilk 2-4 hafta **Denetim** modunda çalıştırılmalı, ürettiği loglar incelenmeli ve yanlış pozitif tespiti yapılmalıdır. Ardından güvenli şekilde **Engelle** moduna alınabilir. Tüm kuralları aynı anda Engelle moduna almak, özellikle Office uygulamalarına bağımlı iş birimlerinde ciddi kesintilere yol açabilir.

---

### 3.4 Microsoft Defender for Cloud Apps — Session, Access ve File Policy

**Session Policy (Oturum İlkesi):**

Kullanıcı ile bulut uygulaması arasına ters proxy olarak giren, oturum içi gerçek zamanlı kontrol sağlar. Oluşturma adımları:

1. security.microsoft.com → Bulut Uygulamaları → İlkeler → İlke Oluştur → Oturum İlkesi.
2. **Şablon seç veya sıfırdan oluştur:** "Hassas belgenin indirilmesini engelle", "Yönetilmeyen cihazdan yüklemeyi izle" gibi hazır şablonlar mevcuttur.
3. **Koşullar:** Hangi uygulama, hangi kullanıcı grubu, cihaz yönetilmiş mi (Intune uyumluluğu CA üzerinden), faaliyet türü (indirme, yükleme, yazdırma, paylaş).
4. **İçerik denetimi:** İçerik denetimi etkinleştirilirse, transfer edilen dosyanın içeriği DLP motoru tarafından taranır (Purview DLP politikası veya yerleşik içerik denetimi); hassas etiket varsa ek eylem devreye girer.
5. **Eylem:** İzin ver, Engelle, Koru (şifreleme etiketini zorla), Denetle.

**Access Policy (Erişim İlkesi):**

Oturum politikasından farklı olarak, uygulamaya ilk girişte (oturum başlamadan) karar verir: izin ver mi, engelle mi? Yönetilemeyen cihazdan tamamen erişimi kesmek veya konuma göre erişimi kısıtlamak için kullanılır; Session Policy ise oturum içi ayrıntılı kontrolü sağlar.

**File Policy (Dosya İlkesi):**

Gerçek zamanlı oturum trafiği değil, SaaS uygulamasındaki (SharePoint, OneDrive, Salesforce, Box gibi) mevcut dosyaları tarayan politikadır. Koşul: hassasiyet etiketi içeriyor mu, harici kullanıcıyla paylaşılmış mı, belirli bir veri deseni (kimlik no, kredi kartı) barındırıyor mu? Eylem: paylaşımı kaldır, etiketi güncelle, güvenlik ekibine bildir.

---

### 3.5 Microsoft Purview — DLP, Sensitivity Label ve Retention Policy

**DLP Policy (Veri Kaybı Önleme İlkesi):**

1. compliance.microsoft.com → Veri Kaybı Önleme → İlkeler → İlke Oluştur.
2. **Şablon veya özel:** Microsoft finansal, sağlık, GDPR gibi hazır şablonlar sunar; kurumun Purview veri sınıflandırma taksonomisine (hassasiyet etiketleri) göre özel ilke tercih edilir.
3. **Konum seçimi:** İlke hangi kanallarda aktif? Exchange (e-posta), SharePoint, OneDrive, Teams, Uç Nokta (cihaz üzerindeki dosya işlemleri — USB, kopyala/yapıştır, yazdırma), Bulut Uygulamaları (üçüncü taraf SaaS). Her kanal ayrı ayrı veya hepsi birlikte seçilebilir.
4. **İçerik koşulları:** Hassasiyet etiketi X içeriyorsa, VEYA SIT (Sensitive Information Type — T.C. kimlik numarası, kredi kartı deseni, IBAN) içeriyorsa. SIT'ler güven eşiğiyle (confidence level) ayarlanır: %75 güvenle eşleşme mi, %95 mi?
5. **Eylem:** Erişimi kısıtla, etkinliği bildir, kullanıcıyı uyar (business justification girişine izin ver), içeriği engelle.
6. **Test modu:** DLP politikaları da önce "Simülasyon" (Test) modunda çalıştırılmalı; üretilen uyarı raporları incelenerek yanlış pozitifler düzeltilmeli, ardından canlıya alınmalıdır.

**Sensitivity Label Policy:**

Hassasiyet etiketlerini tanımlamak (label tanımı) ile bu etiketleri kullanıcılara yayınlamak (label policy) iki ayrı adımdır. Yayınlama politikası, hangi etiketin hangi kullanıcı grubuna görünür olacağını ve varsayılan etiketin ne olacağını belirler. Auto-labeling policy ise içerik analizine göre etiketi kullanıcı müdahalesi olmadan otomatik uygular; bu, uygulama tutarlılığını sağlar.

**Retention Policy ve Label:**

Kapsam + süre + eylem üçlüsüyle tanımlanır: hangi konumdaki içerik (Exchange, SharePoint, OneDrive, Teams kanalları), kaç yıl saklanacak, süre sonunda silinsin mi yoksa gözden geçirilsin mi (disposition review)?

---

### 3.6 Azure Policy — Altyapı Katmanı Zero Trust

Azure Policy, IaaS/PaaS kaynaklarının tanımlı güvenlik standardından sapmasını (drift) önler veya tespit eder. Zero Trust için kritik Azure Policy senaryoları:

- SQL Server'da "Kimlik Doğrulama Modu" yalnızca Azure AD (Entra ID) olmalı — yerel SQL kimlik doğrulamasını yasaklar.
- Sanal makinelerde disk şifreleme (Azure Disk Encryption) zorunlu.
- Tüm depolama hesapları HTTPS zorunlu, public erişim kapalı.
- Network Security Group (NSG) olmadan hiçbir alt ağ oluşturulmasın.
- Kaynaklar üzerindeki tanılama ayarları (diagnostic settings) Log Analytics workspace'e yönlendirilmiş olmalı.

**Policy Initiative (Girişim):** Birbiriyle ilgili birden fazla policy, bir initiative (girişim) altında gruplanır ve tek bir atama noktasından yönetilir. Microsoft Cloud Security Benchmark initiative'i, hızlı başlangıç için hazır bir politika paketi sunar.

**Effect (Etki) Seviyeleri — Önem sırası:** Disabled → Audit → AuditIfNotExists → Append → Modify → Deny → DeployIfNotExists. Başlangıç için Audit, stabilizasyon sonrası Deny önerilir — özellikle büyük Azure ortamlarında Deny etkili politikalar canlı iş yüklerini kırabilir.

---

### 3.7 Microsoft Sentinel — Analytics Rule ve Automation Rule

**Analytics Rule (Analitik Kural):**

Tüm politika nesneleri içinde en esnek ve teknik olanıdır; KQL (Kusto Query Language) ile yazılır. Yapısı:

1. **KQL sorgusu:** Hangi log tablosunda, hangi koşullarla (EventType, Severity, SinyalKaynağı) arama yapılsın?
2. **Zamanlama:** Her 5 dakikada mı, saatte mi, günde mi değerlendirilsin? Kaç günlük veri bakılsın?
3. **Olay oluşturma eşiği:** Sorgu kaç eşleşmede tetiklensin? Her eşleşmede mi, belirli bir pencerede belirli sayıda eşleşmede mi?
4. **Varlık eşleştirme:** Olaydaki Hesap, IP, Cihaz gibi varlıklar Sentinel'in UEBA veri tabanıyla eşleştirilerek risk puanı zenginleştirilir.
5. **MITRE ATT&CK etiketleri:** Kuralın hangi saldırı tekniğini (taktik/teknik) kapsadığını belirler; bu, tehdit kapsam haritası oluşturmak için kullanılır.

**Automation Rule (Otomasyon Kuralı):**

Playbook'ların (Logic Apps) hangi olayda tetikleneceğini tanımlar. Önkoşul — yüksek öncelikli analitik kural tarafından oluşturulan bir olay, bir otomasyon kuralıyla ilgili playbook'u çalıştırır (örneğin: "Yüksek öncelikli kimlik ihlali olayı → Kullanıcıyı Entra ID'den otomatik devre dışı bırak playbook'unu çalıştır"). Otomasyon kuralları, aynı playbook'u birden fazla analitik kurala bağlamayı kolaylaştırır.

---

## 4. Politika Değerlendirme Sırası ve Çakışma Yönetimi

Çakışma yönetimi, özellikle çok sayıda CA politikası veya Intune profili olan kurumlarda kritik bir operasyonel konudur:

**Entra ID Conditional Access'te çakışma:** Birden fazla politika aynı kullanıcı/oturum için geçerliyse, tümü paralel değerlendirilir. En kısıtlayıcı politika kazanır; "Engelle" her zaman "İzin ver"i geçersiz kılar. Bu nedenle "tüm kullanıcılar + tüm uygulamalar" kapsamında bir Block politikası varsa, diğer tüm politikalar anlamsızlaşır — kapsam tanımı ve hariç tutmalar son derece dikkatli yapılmalıdır.

**Intune'da çakışma:** Aynı cihaza birden fazla Configuration Profile atandığında ve aynı ayar için farklı değerler içerdiklerinde, "en güvenli değer" veya "son yazılan kazanır" mantığı ayara göre değişir. Çakışmalar Intune'un "Cihaz Durumu → Profil Durumu" raporunda görünür ve elle çözümlenmesi gerekir. Bu nedenle profil tasarımında kapsam grubu çakışmalarını önleyen bir grup yapısı (örn. cihaz türü bazlı dinamik gruplar) önerilir.

**Azure Policy'de öncelik:** Policy atamaları abonelik, yönetim grubu veya kaynak grubu seviyesinde yapılabilir; daha spesifik kapsam (kaynak grubu) genellikle daha geniş kapsamı (abonelik) override eder, ancak bu "exemption" (muafiyet) mekanizmasıyla da yönetilebilir.

---

## 5. Halkalı Dağıtım (Ring Deployment) Stratejisi

Her politika türü için önerilen dağıtım halkası yapısı şudur:

**Halka 0 — Pilot Gurubu:** Güvenlik/BT ekibinden gönüllü 10-20 kullanıcı. Tüm yeni politikalar ilk bu grupta test edilir.

**Halka 1 — Erken Benimseyen Departman:** Güvenlik açısından az kritik bir iş birimi (örn. pazarlama). Pilot'tan gelen geri bildirimlere göre politika rafine edilir.

**Halka 2 — Geniş Dağıtım:** Tüm çalışanların %50-75'i. Yüksek hacimli gerçek kullanım verisine göre son ayarlamalar yapılır.

**Halka 3 — Tam Dağıtım:** Tüm kapsam, break-glass hesapları hariç.

Her halka geçişinde minimum 1-2 hafta beklenmeli ve uyumluluk/yanlış pozitif raporları incelenmelidir.

---

## 6. Break-Glass Hesapları: Politikaların Zorunlu İstisnası

Break-glass (acil erişim) hesapları, tüm politika dağıtımlarının en önemli güvenlik önlemidir. Bu hesaplar, kimlik sağlayıcısının (Entra ID) kendisinde bir sorun çıktığında sisteme acil erişimi garantiler. Özellikler: bulut-yerel hesaplar (federe veya hibrit kimliğe bağımlı değil), güçlü parola (FIDO2 anahtarla), MFA'dan muaf (Conditional Access hariç tutma) ancak bu muafiyetin kendisi loglanmalı ve alertle izlenmeli. Kural: her yeni CA politikasında break-glass grubu **hariç tutulmalıdır**; politika hatasında tüm sisteme erişim kilitlenme riski ortadan kalkar.

---
## 7. Policy as Code Pratiği: Uçtan Uca Kurumsal Uygulama

### 7.1 Neden Policy as Code — Yönetici Gerekçesi

Konsoldan elle yönetilen politikalar üç kronik soruna yol açar: **görünmezlik** (kim, ne zaman, neyi değiştirdi sorusuna net cevap yoktur — Entra ID denetim günlüğü değişikliği gösterir ama "neden" sorusunu cevaplamaz), **tutarsızlık** (test/üretim kiracıları arasında veya bölgeler arasında zamanla sapma oluşur) ve **kurtarma zorluğu** (yanlış yapılandırılmış bir politika geri alınmak istendiğinde, önceki halinin tam olarak ne olduğu çoğu zaman bilinmez). Policy as Code (PaC), her politika nesnesini bir kod deposunda (Git) sürüm kontrollü, gözden geçirmeye (pull request) tabi ve otomatik test edilebilir bir tanım haline getirerek bu üç sorunu da çözer. Yönetim için somut fayda, bir güvenlik olayı sonrası "üç ay önce bu politika neye benziyordu ve kim değiştirdi" sorusuna saniyeler içinde cevap verilebilmesidir.

### 7.2 Repo Yapısı ve Klasörleme Stratejisi

Olgun bir PaC deposu, ürün ve ortam bazında ayrıştırılmış bir yapı izler:

```
zero-trust-policy-as-code/
├── conditional-access/
│   ├── modules/                 # Yeniden kullanılabilir CA politika şablonları
│   ├── environments/
│   │   ├── dev-tenant/
│   │   ├── test-tenant/
│   │   └── prod-tenant/
│   └── tests/                   # Pester/Conftest test dosyaları
├── intune/
│   ├── compliance-policies/
│   ├── configuration-profiles/
│   └── security-baselines/
├── purview-dlp/
│   └── policies/
├── sentinel/
│   ├── analytics-rules/
│   └── automation-rules/
├── azure-policy/
│   ├── definitions/
│   └── initiatives/
├── pipelines/                   # CI/CD tanımları (YAML)
└── docs/
    └── policy-decision-records/ # Her politika kararının gerekçesi (ADR formatı)
```

**Policy Decision Record (PDR):** Her önemli politika değişikliği için, neden bu kararın alındığını açıklayan kısa bir markdown kaydı (Architecture Decision Record pratiğinin politika versiyonu) tutulması önerilir — örneğin "neden break-glass grubu 2 hesapla sınırlandı, neden 5 değil" gibi kararların gerekçesi, altı ay sonra ekip değiştiğinde kaybolmaz.

### 7.3 Araç Seti

| Politika Türü | Önerilen Araç | Notlar |
|---|---|---|
| Conditional Access | Terraform (`azuread` / `azuread` provider) veya Microsoft Graph PowerShell SDK | Terraform, state yönetimi ve plan/apply döngüsü sunar |
| Intune (Compliance/Config/Baseline) | **Microsoft365DSC** (PowerShell DSC tabanlı) | Microsoft'un resmi olarak desteklediği, M365/Intune'a özel kapsamlı kaynak seti |
| Purview DLP / Sensitivity Label | Microsoft365DSC veya Graph PowerShell SDK | DLP kuralları karmaşık koşul ağaçları içerebileceğinden test kapsamı genişletilmeli |
| Azure Policy / Initiative | **Bicep** veya Terraform (`azurerm_policy_definition`) | Bicep, Azure-yerel kaynaklar için ARM şablonlarına göre daha okunur sözdizimi sunar |
| Sentinel Analytics/Automation Rules | Bicep (ARM şablon tabanlı, Sentinel'in resmi "Solutions" deposu formatı) | Microsoft'un Sentinel-All-In-One GitHub deposu referans alınabilir |
| Genel doğrulama / politika testi | **OPA (Open Policy Agent) / Conftest** | Çıktı JSON/Bicep planlarının kurumsal standartlara (örn. "her CA politikasında break-glass exclude var mı") uyup uymadığını statik olarak doğrular |

### 7.4 Ortam Stratejisi: Dev → Test → Prod Kiracı Ayrımı

PaC'nin güvenilir çalışması için en az üç ayrı Entra ID kiracısı (veya en azından izole edilmiş bir test ortamı) önerilir:

- **Dev Kiracı:** Geliştiricilerin serbestçe deneme yapabildiği, üretim verisinden tamamen izole ortam.
- **Test/Staging Kiracı:** Üretime yakın yapılandırma, pilot halka (Bölüm 5'teki ring deployment) burada doğrulanır.
- **Prod Kiracı:** Gerçek kullanıcı ve kaynakların bulunduğu canlı ortam; yalnızca onaylı pipeline üzerinden değişiklik kabul eder, manuel konsol değişikliği mümkünse kısıtlanır (Entra ID'de "Restrict non-admin users from making changes" benzeri kontrollerle desteklenir).

Tek kiracılı küçük kurumlarda tam ortam ayrımı pratik olmayabilir; bu durumda en azından **Report-only modu** (CA için) ve **Audit etkisi** (Azure Policy için) bir "sanal test ortamı" işlevi görür — kod aynı pipeline'dan geçer ama prod kiracıda önce denetim modunda uygulanır.

### 7.5 CI/CD Pipeline Tasarımı

Tipik bir pipeline beş aşamadan oluşur:

1. **Lint & Statik Analiz:** Terraform `fmt`/`validate`, Bicep `build`, PowerShell Script Analyzer. Söz dizimi hataları burada yakalanır.
2. **Policy Validation (OPA/Conftest):** Kurumsal kurallara uyum kontrolü — örneğin "her yeni CA politikasında `excludeGroups` alanı break-glass grup ID'sini içermeli", "her Azure Policy ataması `enforcementMode: DoNotEnforce` ile başlamalı (ilk dağıtımda)". Bu aşama, Bölüm 6'daki break-glass kuralının kod seviyesinde otomatik zorunlu kılınmasıdır.
3. **Plan / What-If:** Terraform `plan` veya Azure `what-if` komutu, değişikliğin gerçek etkisini (kaç kaynak/politika etkilenecek) hesaplar ve pull request'e otomatik yorum olarak eklenir — gözden geçiren kişi, "bu değişiklik 4.200 kullanıcıyı etkileyecek" bilgisini onaydan önce görür.
4. **Onay Kapısı (Approval Gate):** En az bir güvenlik mühendisi onayı zorunlu (CODEOWNERS dosyasıyla zorlanır); prod ortamına yönelik değişikliklerde iki onay önerilir. Yüksek riskli değişiklikler (örn. `state: enabled` olan bir Block politikası) için ek bir manuel onay adımı eklenebilir.
5. **Apply & Doğrulama:** Onay sonrası otomatik uygulama; ardından bir doğrulama adımı (smoke test) çalışır — örneğin yeni bir CA politikası uygulandıktan sonra, break-glass hesabının hâlâ oturum açabildiği otomatik test edilir.

### 7.6 Pipeline Kimliği: Workload Identity Federation ile Sırsız CI/CD

Pipeline'ın kendisinin Entra ID/Azure'a kimlik doğrulaması yapması gerekir; bu noktada Bölüm 3'te ele alınan **Workload Identity Federation** doğrudan devreye girer. GitHub Actions veya Azure DevOps pipeline'ı, uzun ömürlü bir client secret saklamak yerine OIDC token değişimiyle kısa ömürlü erişim token'ı alır — bu, "policy as code sistemi sırsız çalışır" prensibinin somut uygulamasıdır ve PaC altyapısının kendisinin bir saldırı yüzeyi haline gelmesini önler.

```yaml
# GitHub Actions — Azure'a OIDC ile (sırsız) kimlik doğrulama örneği
name: policy-as-code-deploy
on:
  pull_request:
    branches: [main]
permissions:
  id-token: write   # OIDC token talebi için zorunlu
  contents: read
jobs:
  plan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
          # not: client-secret YOK — federated credential kullanılıyor
      - run: terraform init && terraform plan -out=tfplan
      - run: conftest test tfplan.json --policy ./opa-rules/
```

Pipeline'a atanan service principal'ın izinleri de en az ayrıcalık ilkesiyle sınırlanmalı: yalnızca `Policy.ReadWrite.ConditionalAccess` gibi spesifik Graph API izinleri verilmeli, Global Administrator gibi geniş roller asla pipeline kimliğine atanmamalıdır.

### 7.7 Test Stratejileri

**Birim testleri (Pester — PowerShell):** Microsoft365DSC tabanlı tanımlar için, "bu yapılandırma bloğu beklenen JSON şemasını üretiyor mu" düzeyinde testler.

**Politika doğrulama testleri (OPA/Conftest):** Kurumsal standart kontrolleri — aşağıda örnek bir Rego kuralı, her CA Block politikasının mutlaka break-glass grubunu hariç tuttuğunu doğrular:

```rego
package policy.conditional_access

deny[msg] {
  input.resource_type == "conditional_access_policy"
  input.grant_controls.built_in_controls[_] == "block"
  not contains(input.conditions.users.exclude_groups, "<break-glass-grup-id>")
  msg := "Block etkili CA politikası break-glass grubunu hariç tutmalı"
}
```

**Entegrasyon testleri (test kiracısında):** Pipeline, test kiracısında gerçek bir politika uygulayıp, ardından Graph API ile "bu politika gerçekten oluştu mu, beklenen koşulları içeriyor mu" diye doğrular; ardından temizlik (cleanup) yapar.

**Report-only / Audit doğrulaması:** Yeni bir CA politikası prod'a ilk girişte daima `enabledForReportingButNotEnforced` durumunda dağıtılır; pipeline bir sonraki aşamada (örn. 2 hafta sonra, ayrı bir manuel onaylı pipeline çalıştırmasıyla) `enabled` durumuna geçirir. Bu iki aşamalı dağıtım, Bölüm 5'teki halkalı dağıtımın kod seviyesinde zorunlu kılınmasıdır.

### 7.8 Çoklu Politika Türü İçin Kod Örnekleri

**Conditional Access — Terraform (`azuread` provider):**

```hcl
resource "azuread_conditional_access_policy" "zt_mfa_all_users" {
  display_name = "ZT-Faz1-MFA-Tum-Kullanicillar"
  state        = "enabledForReportingButNotEnforced"

  conditions {
    client_app_types    = ["all"]
    sign_in_risk_levels = []
    user_risk_levels    = []

    applications {
      included_applications = ["All"]
    }
    users {
      included_users  = ["All"]
      excluded_groups = [var.break_glass_group_id]
    }
  }

  grant_controls {
    operator          = "OR"
    built_in_controls = ["mfa"]
  }
}
```

**Intune Compliance Policy — Microsoft365DSC:**

```powershell
Configuration ZT_Intune_Compliance {
    Import-DscResource -ModuleName 'Microsoft365DSC'

    IntuneDeviceCompliancePolicyWindows10 'ZT-Windows-Uyumluluk' {
        DisplayName                       = 'ZT-Faz2-Windows-Uyumluluk'
        Ensure                            = 'Present'
        BitLockerEnabled                  = $true
        SecureBootEnabled                 = $true
        OsMinimumVersion                  = '10.0.19045'
        ActiveFirewallRequired            = $true
        DefenderEnabled                   = $true
        ScheduledActionsForRule           = @(
            MSFT_DeviceManagementComplianceScheduledActionForRule {
                GracePeriodHours = 24
                ActionType       = 'block'
            }
        )
    }
}
```

**Sentinel Analytics Rule — Bicep:**

```bicep
resource analyticsRule 'Microsoft.SecurityInsights/alertRules@2023-11-01' = {
  name: 'zt-yuksek-risk-anormal-indirme-korelasyonu'
  kind: 'Scheduled'
  properties: {
    displayName: 'Yüksek Riskli Kullanıcı + Anormal İndirme Korelasyonu'
    severity: 'High'
    enabled: true
    query: '''
      let riskyUsers = SigninLogs | where RiskLevelDuringSignIn == "high" | distinct UserPrincipalName;
      CloudAppEvents
      | where ActionType == "FileDownloaded"
      | where AccountObjectId in (riskyUsers)
    '''
    queryFrequency: 'PT15M'
    queryPeriod: 'PT1H'
    triggerOperator: 'GreaterThan'
    triggerThreshold: 0
  }
}
```

**Azure Policy — Bicep:**

```bicep
resource diskEncryptionPolicy 'Microsoft.Authorization/policyAssignments@2022-06-01' = {
  name: 'zt-disk-encryption-zorunlu'
  properties: {
    displayName: 'ZT-AzureVM-DiskEncryption'
    policyDefinitionId: subscriptionResourceId(
      'Microsoft.Authorization/policyDefinitions',
      '<built-in-policy-id>'
    )
    enforcementMode: 'DoNotEnforce' // İlk dağıtımda Audit; stabilizasyon sonrası 'Default'a çevrilir
  }
}
```

### 7.9 Drift Tespiti ve Otomatik Reconciliation

Konsoldan yapılan manuel bir acil müdahale (örn. bir kesinti sırasında bir mühendisin CA politikasını elle değiştirmesi), kod tanımıyla canlı durum arasında sapma (drift) yaratır. Olgun bir PaC pratiği bu sapmayı tolere etmez, **tespit eder ve raporlar**:

- Günlük/haftalık zamanlanmış bir pipeline çalışması, `terraform plan` (veya Bicep `what-if`) komutunu sıfır değişiklikle (no-op olması beklenerek) çalıştırır.
- Plan, beklenmedik bir fark gösterirse (yani biri konsoldan elle değişiklik yapmışsa), bu otomatik olarak Sentinel'e veya bir Teams/e-posta kanalına bir "policy drift detected" uyarısı olarak düşer.
- Drift, ya kod tanımına geri alınır (`terraform apply` ile reconciliation) ya da meşru bir acil durum değişikliğiyse, kod tanımı güncellenip bir pull request ile resmileştirilir — **hiçbir konsol değişikliği kalıcı olarak kod dışında bırakılmaz.**

### 7.10 Rollback Stratejisi

Git geçmişi, doğası gereği bir rollback mekanizmasıdır: önceki bir commit'e dönülüp pipeline yeniden çalıştırılarak politika önceki haline döndürülebilir. Ancak iki ek önlem önerilir: **Terraform state'in (veya Bicep dağıtım geçmişinin) şifrelenmiş, sürüm kontrollü bir uzak depoda (Azure Storage + sürüm kontrolü etkin) tutulması** — state kaybı, kod ile gerçek durum arasındaki bağlantıyı tamamen koparır; **her prod `apply` işleminden önce otomatik bir yedek (snapshot) alınması** — özellikle Intune/DLP gibi Microsoft365DSC ile yönetilen alanlarda, dağıtım öncesi mevcut durumun JSON dökümünün pipeline tarafından otomatik saklanması, en kötü senaryoda dakikalar içinde manuel geri yüklemeye imkân tanır.

### 7.11 PaC Olgunluk Modeli

| Seviye | Tanım |
|---|---|
| 0 — Manuel | Tüm politikalar konsoldan elle yönetilir, değişiklik geçmişi yalnızca denetim günlüğünde. |
| 1 — Dışa Aktarılan Kod | Mevcut politikalar periyodik olarak script ile dışa aktarılıp Git'e yedeklenir (salt-okunur, henüz dağıtım kod üzerinden yapılmıyor). |
| 2 — Kod ile Dağıtım | Yeni politikalar kod üzerinden (Terraform/Bicep/M365DSC) oluşturulur, ancak test/onay süreci henüz tam otomatik değil. |
| 3 — CI/CD ile Onaylı Dağıtım | Tüm değişiklikler pull request + otomatik plan/test + onay kapısı sürecinden geçer; pipeline kimliği OIDC ile sırsız çalışır. |
| 4 — Tam GitOps + Otomatik Reconciliation | Drift tespiti otomatik, sapmalar otomatik raporlanır/düzeltilir; politika deposu, canlı ortamın tek doğruluk kaynağı (single source of truth) olarak kabul edilir. |

Çoğu kurum için gerçekçi hedef, Faz 2-3 sürecinde Seviye 2'ye, Faz 4'e (Optimal) ulaşıldığında ise en az kritik politika kümeleri (CA, Azure Policy, Sentinel) için Seviye 3-4'e ulaşmaktır.

---

## 8. Politika İzleme ve Denetim

Her politika nesnesi, ayrı izleme kanallarına sahiptir; bunların Sentinel'e beslenmesi Bölüm 11'de detaylandırılmış olup aşağıda hangi politika türünün nerede izleneceği özetlenmektedir:

- **CA politikaları:** Entra ID → Oturum Açma Günlükleri. Her oturum için hangi politikaların uygulandığı, hangi etkinin devreye girdiği görülebilir. "Report-only" modunda bile oturum günlükleri politikanın teorik etkisini raporlar.
- **Intune uyumluluk:** Intune portalı → Cihazlar → Uyumluluk raporları. Uyumsuz cihaz listesi, uyumsuzluk nedeni (hangi ayar eksik), zaman içindeki trend.
- **DLP politikaları:** Purview portalı → Veri Kaybı Önleme → Uyarılar ve Activity Explorer. Hangi kullanıcı, hangi dosyada, hangi kanalla DLP kuralını tetikledi?
- **Azure Policy:** portal.azure.com → Policy → Uyumluluk. Kaynak ve politika bazlı uyumluluk yüzdeleri.
- **Sentinel analytics kuralları:** Sentinel → Analiz → Kural durumu, üretilen olay sayısı, yanlış pozitif oranı.

---

## 9. Tek Sinyal, Çok Politika: Entegrasyon Zinciri

Microsoft policy ekosisteminin asıl Zero Trust gücü, **tek bir sinyalin birden fazla policy nesnesini art arda besleyebilmesidir.** Bütünleşik bir örnek:

> Senaryo: Bir kullanıcının kimlik bilgileri sızdırılmış ve hesabı ele geçirilmiş.

1. Entra ID Protection → kullanıcı riski "Yüksek" olarak işaretlendi (**sinyal üretimi**).
2. CA Policy "Yüksek Riskli Kullanıcı → Parola Değiştir" → kullanıcıyı her uygulamada parola sıfırlamaya yönlendirdi (**kimlik katmanı politika etkisi**).
3. Defender for Cloud Apps Session Policy → aynı oturumda hassas dosya indirme girişimini engelledi (**uygulama katmanı politika etkisi**).
4. Sentinel Analytics Rule → "Yüksek riskli kullanıcı oturumu + anormal indirme girişimi" korelasyonuyla yüksek öncelikli olay oluşturdu (**çapraz sinyal politika etkisi**).
5. Sentinel Automation Rule → playbook tetiklendi, kullanıcının tüm aktif oturumları CAE ile sonlandırıldı, güvenlik ekibi bildirildi (**otomatik müdahale politika etkisi**).

Beş farklı politika nesnesi, birbiriyle konuşmadan değil, **ortak bir sinyal omurgası üzerinden** zincirleme çalıştı.

---

## Faz 0 Tamamlanma Kriterleri (Policy Odaklı)

- Kullanılacak her Microsoft ürünü için hangi policy nesnesinin hangi Zero Trust sütununu karşıladığı belgelenmiş olmalı.
- Tüm yeni politikalar için "önce Report-only/Test, sonra pilot halka, sonra tam dağıtım" standart prosedürü tanımlanmış olmalı.
- Break-glass grubu oluşturulmuş ve tüm CA politikalarında hariç tutma olarak eklenmiş olmalı.
- Kritik CA ve Azure Policy nesneleri için en az elle dışa aktarım (export) tabanlı sürüm kontrolü başlatılmış olmalı.
- Policy as Code olgunluk modelinde kurumun mevcut seviyesi (0-4) belirlenmiş ve en az Seviye 2'ye (kod ile dağıtım) geçiş planı çıkarılmış olmalı; pipeline kimliği için Workload Identity Federation kullanılacak şekilde tasarlanmış olmalı.
- Politika uyumluluğunu izleyen temel raporlar (Intune uyumluluk, CA oturum açma, Azure Policy uyumluluk) düzenli gözden geçirme takvimine alınmış olmalı.

## Sonraki Bölüm

Bölüm 3: **Microsoft Entra ID ile Kimlik Temeli** — bu policy çerçevesinin ilk ve en kritik uygulama alanı olan kimlik temelini, Conditional Access politika tasarımını merkeze alarak ele alacağız.
