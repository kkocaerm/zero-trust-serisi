---
layout: default
title: "Microsoft Entra ID ile Kimlik Temeli"
parent: "Faz 1: Kimlik Temeli"
nav_order: 1
---

# Bölüm 3 (Faz 1): Microsoft Entra ID ile Kimlik Temeli — İnsan ve İnsan Dışı Kimlikler, API ve Anahtar Güvenliği

## Yönetici Özeti

Zero Trust mimarisinde kimlik "yeni çevre"dir (the new perimeter). Ancak çoğu kurum "kimlik" derken yalnızca çalışanları ve onların kullanıcı adı/şifre çiftini düşünür. Gerçekte, modern bir kurumsal kiracıda **insan dışı kimlikler (Non-Human Identities — NHI)** — servis hesapları, uygulama kimlikleri, API'ler, otomasyon script'leri, CI/CD pipeline'ları — sayıca insan kimliklerini çoktan geçmiştir; sektör verilerine göre büyük kurumlarda bu oran 10:1 ila 50:1 arasına çıkabilmektedir. Bu kimlikler genellikle kalıcı parolalarla/anahtarlarla çalışır, asla rotate edilmez, kimse onların sahibini bilmez ve çoğu zaman aşırı yetkilendirilmiştir — yani saldırganlar için MFA korumalı bir kullanıcı hesabından çok daha kolay bir hedeftir.

Bu bölüm, Bölüm 2.5'te kurduğumuz Microsoft Policy çerçevesini insan kimliklerinin ötesine taşıyarak şu dört alanı ele alır: **hibrit kimlik ve insan kimliği temeli** (orijinal kapsam), **insan dışı kimlikler** (servis hesabı yayılması, makine kimlikleri, bulut ayrıcalık zincirleme, kimlik grafiği güvenliği), **API güvenliği** ve **anahtar/sır güvenliği**. Yönetim için mesaj açıktır: Zero Trust programı, yalnızca çalışan kimliklerini MFA ile korumakla "kimlik sütunu tamamlandı" diyemez; görünmeyen, sayıca daha büyük ve genellikle denetimsiz olan makine kimlikleri kapsanmadan bu sütun olgunlaşmamış sayılır.

---

## Bölüm A: İnsan Kimliği Temeli

### Hibrit Kimlik Mimarisi

Çoğu kurum, on-premises Active Directory'den tamamen vazgeçmeden Entra ID'ye geçiş yapar. Bunun için iki temel senkronizasyon yöntemi vardır:

**Entra Connect Sync (Password Hash Sync):** AD'deki parola özetleri Entra ID'ye senkronize edilir; kimlik doğrulama bulutta gerçekleşir. Çoğu kurum için önerilen yaklaşımdır, çünkü on-prem bağımlılığını azaltır ve Entra ID Protection'ın risk tespiti özelliklerinden tam olarak yararlanılabilir.

**Pass-through Authentication (PTA) veya Federasyon (AD FS):** Kimlik doğrulama on-prem'de kalır. Bu yaklaşım, on-prem altyapıya bağımlılığı korur ve Zero Trust'ın "bulut-öncelikli, sürekli değerlendirme" hedefiyle daha az uyumludur; yalnızca düzenleyici gereklilikler zorunlu kılıyorsa tercih edilmelidir.

### MFA Zorunluluğu ve Güvenlik Varsayılanları

Faz 1'in ilk hızlı kazanımı, tüm kullanıcılar için MFA'nın zorunlu kılınmasıdır:

1. **Güvenlik Varsayılanları (Security Defaults):** Lisanstan bağımsız, tüm kullanıcılar için temel MFA ve eski kimlik doğrulama protokollerinin engellenmesini sağlayan tek tıkla açılan bir özellik.
2. **Conditional Access tabanlı MFA (Entra ID P1/P2):** Bölüm 2.5'te detaylandırılan, koşullara göre (risk, konum, uygulama) farklılaştırılmış MFA politikaları.
3. **Parolasız Kimlik Doğrulama:** FIDO2 güvenlik anahtarları, Windows Hello for Business, Microsoft Authenticator ile telefon tabanlı parolasız oturum açma. Optimal olgunluk seviyesinde hedef budur.

### Eski Kimlik Doğrulama Protokollerinin Kapatılması

POP3, IMAP, SMTP AUTH gibi eski protokoller MFA'yı desteklemez ve saldırganlar tarafından MFA'yı atlamak için sıklıkla hedef alınır. Bu protokollerin Entra ID üzerinde kurum genelinde devre dışı bırakılması, tek bir politika değişikliğiyle saldırı yüzeyini önemli ölçüde küçültür.

### Kimlik Yaşam Döngüsü Yönetimi

- **Otomatik provizyon/de-provizyon:** İK sisteminden Entra ID'ye otomatik kullanıcı oluşturma ve işten çıkışta anında erişim kaldırma (Lifecycle Workflows).
- **Hesap hijyeni:** Kullanılmayan, sahipsiz veya aşırı yetkili hesapların düzenli taranması.

---

## Bölüm B: İnsan Dışı Kimlikler (Non-Human Identities)

### B.1 Servis Hesabı Yayılması (Service Account Sprawl)

**Sorun tanımı:** Yıllar içinde her yeni entegrasyon, otomasyon script'i veya zamanlanmış görev için "hızlıca" bir servis hesabı oluşturulur. Bu hesaplar genellikle:

- Asla süresi dolmayan (parola politikasından muaf) statik parolalarla çalışır.
- Gereğinden fazla yetkiyle (çoğu zaman Domain Admin veya Global Administrator) oluşturulur, çünkü "çalışsın da hangi izin gerektiğini sonra ayıklarız" yaklaşımı yaygındır.
- Sahibi kurumdan ayrıldığında hiç kimsenin sorumlu olmadığı "yetim" (orphaned) hesaplara dönüşür.
- MFA ve Conditional Access politikalarından çoğu zaman muaf tutulur, çünkü "otomasyonu bozmasın" denir — bu da onları en az korunan ayrıcalıklı hesaplara çevirir.

**Tespit ve envanter:** Entra ID → Kimlikler → Uygulamalar → Kurumsal Uygulamalar listesi ve PowerShell/Graph API ile son oturum açma tarihi (`signInActivity`) sorgulanarak 90+ gün kullanılmamış servis hesapları/service principal'lar tespit edilir. Microsoft Entra ID Governance'taki **Access Reviews**, service principal sahipliğini de kapsayacak şekilde yapılandırılabilir.

**Düzeltme stratejisi:**

1. Her servis hesabı/service principal'a bir **iş sahibi (owner)** atanmalı — sahipsiz hesap, bulunduğu anda devre dışı bırakılmaya aday olmalı.
2. Statik parola/anahtar kullanan klasik servis hesapları, mümkün olduğunda **Managed Identity** (Bölüm B.2) ile değiştirilmeli.
3. Tamamen kaldırılamayan eski servis hesapları için, en az ayrıcalık ilkesiyle rol ataması yeniden gözden geçirilmeli ve mümkünse parola/anahtar rotasyonu otomatikleştirilmeli (Azure Key Vault rotasyon politikası — Bölüm D).
4. On-prem AD'de **Group Managed Service Account (gMSA)** kullanımı, klasik statik parolalı servis hesaplarının yerini almalı; gMSA parolaları otomatik ve periyodik olarak AD tarafından rotate edilir ve insan eliyle yönetilmez.

### B.2 Makine Kimlikleri (Machine Identities)

Makine kimlikleri, Entra ID'de iki temel nesneyle temsil edilir:

**Service Principal:** Bir uygulamanın belirli bir kiracıdaki kimlik temsilidir; uygulama kayıtlarının (App Registration) kiracı içindeki "yerel kopyası" gibi düşünülebilir. API izinleri (Bölüm C) ve rol atamaları bu nesneye bağlanır.

**Managed Identity (Yönetilen Kimlik):** Azure kaynaklarının (VM, App Service, Function, AKS pod) kimlik bilgisi saklamadan Entra ID'de kimlik doğrulaması yapmasını sağlayan mekanizmadır — **bu, makine kimlikleri için Zero Trust'ın hedef durumudur**, çünkü ortada saklanması/sızdırılması gereken bir sır (secret) yoktur; kimlik bilgisi platform tarafından otomatik yönetilir ve rotate edilir.

- **Sistem Tarafından Atanan (System-assigned):** Kaynağın yaşam döngüsüne bağlıdır; kaynak silindiğinde kimlik de silinir. Tek bir kaynağa özgü kullanımlar için uygundur.
- **Kullanıcı Tarafından Atanan (User-assigned):** Bağımsız bir kaynak olarak oluşturulur, birden fazla Azure kaynağına atanabilir; paylaşılan kimlik gerektiren senaryolarda (örn. aynı AKS cluster'ındaki birden fazla pod) tercih edilir, ancak paylaşım arttıkça "blast radius" (bir kimliğin ele geçirilmesi durumunda etkilenen kaynak sayısı) da büyür — bu nedenle paylaşım kapsamı bilinçli sınırlanmalıdır.

**Workload Identity Federation (İş Yükü Kimlik Federasyonu):** GitHub Actions, GitLab CI/CD veya Kubernetes gibi dış sistemlerin, Entra ID'de bir client secret saklamadan, OIDC token değişimi yoluyla kısa ömürlü erişim token'ı almasını sağlar. CI/CD pipeline'larında uzun ömürlü servis hesabı parolaları yerine workload identity federation kullanılması, Faz 1'in makine kimliği için en yüksek getirili hızlı kazanımlarından biridir.

**Entra ID Workload Identities Premium / Conditional Access for Workload Identities:** Service principal'lar için de Conditional Access politikası tanımlanabilir hale gelmiştir — örneğin bir service principal'ın yalnızca bilinen bir IP aralığından (örn. CI/CD sağlayıcısının IP'leri) erişim talep etmesi zorunlu kılınabilir, anormal konumdan gelen istek engellenir. Ayrıca Entra ID Protection, artık **workload identity risk tespiti** sunar: sızdırılmış bir client secret veya anormal token kullanım deseni, kullanıcı riskine benzer şekilde işaretlenir.

### B.3 Bulut Ayrıcalık Zincirleme (Cloud Privilege Chaining)

**Kavram:** Bir saldırgan, doğrudan yüksek ayrıcalıklı bir kimliği ele geçirmek zorunda değildir; düşük ayrıcalıklı bir kimlikten başlayıp, kaynak/rol ilişkilerini zincirleyerek (chaining) adım adım daha yüksek ayrıcalığa ulaşabilir. Tipik bir zincir örneği:

> Saldırgan, "Contributor" rolüne sahip (görece düşük riskli görünen) bir kullanıcı/service principal'ı ele geçirir → Contributor rolü, bir Sanal Makinede komut çalıştırma (Run Command) yetkisi de içerir → VM üzerinde komut çalıştırarak, o VM'e atanmış Managed Identity'nin erişim token'ını VM'in metadata servisinden çeker → Bu Managed Identity'nin bir Key Vault'a "Get Secret" izni olduğunu görür → Key Vault'tan, çok daha yüksek ayrıcalıklı bir servis hesabının kimlik bilgilerini çeker → Bu kimlik bilgisiyle kritik bir veritabanına veya Global Administrator rolüne sıçrar.

Bu zincirde her tek adım "normal" ve "meşru" görünür; sorun, hiçbir tek rolün aşırı yetkili olmaması ama **rol kombinasyonunun** kritik bir saldırı yolu (attack path) oluşturmasıdır.

**Tespit ve önleme araçları:**

- **Microsoft Defender for Cloud — Attack Path Analysis:** Bulut kaynakları arasındaki ilişkileri (rol atamaları, ağ bağlantıları, kimlik erişimleri) bir grafik olarak modelleyip, internete açık bir kaynaktan en kritik varlığa (örn. Global Admin) uzanan olası saldırı yollarını otomatik olarak görselleştirir ve önceliklendirir.
- **Microsoft Entra Permissions Management (CIEM — Cloud Infrastructure Entitlement Management):** Azure, AWS ve GCP genelinde hangi kimliğin **fiilen kullandığı** ile **sahip olduğu** izinler arasındaki farkı (permission gap) raporlar; "kullanılmayan ayrıcalık" en sık zincirleme malzemesidir ve bu fark küçültülerek saldırı yüzeyi azaltılır.
- **Rol tasarımı:** Geniş yerleşik roller (Contributor, Owner) yerine, gerçekten ihtiyaç duyulan eylemleri içeren özel (custom) roller tanımlanmalı; "Run Command" gibi tehlikeli eylem izinleri, gerçekten gerekmedikçe verilmemelidir.
- **Just-in-Time + PIM genişletmesi:** Bölüm 4'te ele alınan PIM, yalnızca insan kimliklerine değil, mümkün olduğunda yüksek riskli servis prensiplerine de (PIM for Groups ile) uygulanmalıdır.

### B.4 Identity Graph Security (Kimlik Grafiği Güvenliği)

**Kavram:** Kimlikleri tek tek (izole varlıklar olarak) güvenli kılmak yeterli değildir; kullanıcılar, gruplar, roller, kaynaklar ve service principal'lar arasındaki **ilişkiler** bir grafik (graph) oluşturur ve saldırganlar bu grafiği "tek tek kimlik kırmak" yerine "en kısa ayrıcalık yükseltme yolunu bulmak" amacıyla tararlar (bu teknik, on-prem Active Directory dünyasında BloodHound gibi araçlarla popülerleşmiş, bulut kimlik sağlayıcılarında da doğrudan karşılığı bulunan bir saldırı yöntemidir).

Tipik bir kimlik grafiği saldırı yolu sorusu şudur: *"Düşük ayrıcalıklı bir kullanıcıdan Global Administrator'a, kaç ilişki adımıyla ulaşılabilir?"* Örnek zincir: Kullanıcı A → "Yardım Masası Yöneticisi" grubuna üye → bu grup, parola sıfırlama yetkisine sahip → bu yetkiyle B kullanıcısının (bir uygulama sahibinin) parolasını sıfırlayabilir → B kullanıcısı, kritik bir uygulamanın service principal'ına "Sahip" olarak atanmış → bu service principal'ın Graph API'de `Application.ReadWrite.All` izni var → bu izinle yeni bir uygulama kimlik bilgisi (secret) eklenip, o uygulamanın sahip olduğu yüksek ayrıcalıklı API izinleri kullanılabilir.

**Bu görünmeyen ilişki zincirini yönetmek için:**

- **Microsoft Defender for Identity:** On-prem AD ve hibrit ortamda yanal hareket yollarını (lateral movement paths) ve "Tier 0" varlıklara (Domain Controller, PKI sunucuları gibi en kritik varlıklar) uzanan ilişki zincirlerini tespit eder.
- **Microsoft Entra ID — Identity Secure Score ve İlişki Tabanlı Öneriler:** Entra ID, riskli rol atama kombinasyonlarını ve aşırı geniş grup üyeliklerini "Secure Score" önerileri olarak yüzeye çıkarır.
- **Tier 0 / Ayrıcalıklı Erişim Modeli:** Kurumun en kritik kimlik ve altyapı varlıkları (Global Administrator'lar, Entra Connect sunucuları, PKI, Key Vault'lar) ayrı bir "Tier 0" katmanı olarak tanımlanmalı ve bu katmana erişimi olan **her** ilişki yolu (yalnızca doğrudan rol ataması değil, grup üyeliği, sahiplik, delege izin gibi dolaylı yollar dahil) düzenli olarak haritalanmalıdır.
- **Sentinel ile Sürekli Grafik İzleme:** Bölüm 11'de ele alınan UEBA ve varlık eşleştirme yetenekleri, kimlik grafiğindeki anormal yeni ilişkileri (örn. bir kullanıcının aniden yüksek ayrıcalıklı bir gruba eklenmesi) tespit edip korelasyon kurallarına bağlanabilir.

**Yönetim için kritik mesaj:** Kimlik grafiği güvenliği, "her kimlik tek tek güvenli mi?" sorusundan "kimlikler birbirine bağlandığında en kısa ayrıcalık yükseltme yolu nedir?" sorusuna geçiştir — bu, Optimal Zero Trust olgunluğunun kimlik sütunundaki en ileri seviye yetkinliğidir.

---

## Bölüm C: API Güvenliği

Modern kurumsal mimarinin büyük kısmı API'ler aracılığıyla iletişim kurar; bir API, kimlik doğrulaması zayıf bırakıldığında, tüm NGFW/CASB/DLP yatırımlarını atlayan doğrudan bir veri erişim yoludur.

### Uygulama Kaydı ve İzin Modeli

**App Registration:** Her uygulama (API'yi tüketen veya API'yi sunan), Entra ID'de bir uygulama kaydı ile temsil edilir. Bu kayıt, kimlik doğrulama yöntemini (sertifika, client secret veya federated credential — Bölüm B.2), yönlendirme URI'lerini ve istenen API izinlerini tanımlar.

**Delegated Permissions vs Application Permissions:**

- **Delegated (Temsilci) İzinler:** API çağrısı, oturum açmış bir kullanıcı adına yapılır; izin kapsamı o kullanıcının kendi yetkileriyle sınırlıdır. Örnek: `Mail.Read` (oturum açan kullanıcının kendi postası).
- **Application (Uygulama) İzinleri:** API çağrısı, bir kullanıcı bağlamı olmadan, doğrudan uygulamanın kendi kimliğiyle yapılır ve genellikle kiracı genelinde geçerlidir. Örnek: `Mail.Read` uygulama izni, **kiracıdaki tüm kullanıcıların** postasını okuyabilir — bu nedenle uygulama izinleri çok daha yüksek risklidir ve admin onayı (admin consent) gerektirir.

**En az ayrıcalık ilkesi API izinlerinde:** Bir entegrasyon "Mail.ReadWrite.All" gibi geniş bir uygulama izni istiyorsa ama yalnızca belirli bir paylaşılan kutuyu okuması gerekiyorsa, daha kısıtlı bir izin (Exchange Online'da uygulama erişim politikası ile belirli posta kutusuna sınırlama) tercih edilmelidir. Microsoft Graph API izinleri için **Exchange Online Application Access Policy** ve benzeri kaynak-seviyesi kısıtlamalar, kiracı genelindeki "Application" izinlerinin gerçek kapsamını daraltmak için kullanılmalıdır.

**Admin Consent Workflow:** Kullanıcıların kendi başlarına üçüncü taraf uygulamalara izin vermesi (user consent) kapatılmalı veya sıkı sınırlandırılmalı; tüm yüksek riskli izin talepleri, tanımlı bir admin onay iş akışından geçmelidir. Bu, "illicit consent grant" (kullanıcıyı kandırarak meşru görünümlü bir uygulamaya geniş izin verdirme) saldırı tekniğine karşı temel savunmadır.

### Çalışma Zamanı API Koruması

**Microsoft Defender for APIs:** Defender for Cloud ailesinin bir parçası olarak, API Management üzerinden yayınlanan API'lerin çalışma zamanı davranışını izler; anormal istek hacmi, kimlik doğrulama bypass girişimleri, OWASP API Security Top 10 kategorisindeki bilinen saldırı desenlerini (aşırı veri ifşası, kırık nesne seviyesi yetkilendirme — BOLA) tespit eder.

**Azure API Management (APIM) ile Politika Tabanlı Uygulama:** API Gateway seviyesinde, her isteğe Zero Trust kontrolleri uygulanabilir: JWT token doğrulama (`validate-jwt` politikası), hız sınırlama (rate limiting) ile kaba kuvvet/DoS önleme, IP filtreleme, istek/yanıt şema doğrulama (şema dışı alanları reddet), mTLS (mutual TLS) ile servis-servis arası karşılıklı kimlik doğrulama.

**Token Ömrü ve Kapsam (Scope) Disiplini:** API'lere erişen token'lar, mümkün olduğunca kısa ömürlü olmalı ve istenen kapsam (scope) en aza indirilmelidir; bir API anahtarının/token'ın sızması durumunda etki alanını sınırlamanın en etkili yolu budur.

---

## Bölüm D: Anahtar ve Sır Güvenliği (Key & Secrets Security)

### Azure Key Vault: Merkezi Sır Deposu

Parola, API anahtarı, bağlantı dizesi gibi sırların kod içine, yapılandırma dosyasına veya ortam değişkenine gömülmesi (hardcoding), en sık ve en kolay istismar edilen güvenlik açıklarından biridir — bu sırlar genellikle Git geçmişinde, log dosyalarında veya yedeklerde fark edilmeden kalır. Azure Key Vault, üç tür nesneyi merkezi ve denetlenebilir şekilde yönetir:

- **Secrets:** Parolalar, bağlantı dizeleri, API anahtarları.
- **Keys:** Şifreleme/imzalama anahtarları; yüksek hassasiyetli senaryolarda **HSM-destekli (Hardware Security Module)** anahtarlar kullanılarak anahtar materyalinin yazılım seviyesinde dışa çıkarılması engellenir.
- **Certificates:** TLS/istemci kimlik doğrulama sertifikaları; Key Vault, sertifika yenileme sürecini otomatikleştirebilir.

### Erişim Modeli: RBAC vs Access Policy

Key Vault'a erişim iki modelle yönetilebilir: eski **Vault Access Policy** modeli (Key Vault'a özgü, kapsamı sınırlı izin yapısı) ve önerilen **Azure RBAC** modeli (diğer Azure kaynaklarıyla tutarlı, Entra ID rol atamalarıyla entegre, koşullu erişim ve PIM ile birleştirilebilir). Zero Trust olgunluğu için RBAC modeline geçiş ve "Key Vault Yönetici" gibi yüksek ayrıcalıklı rollerin PIM ile just-in-time hale getirilmesi önerilir.

### Sır Yerine Kimlik: Managed Identity Önceliği

Bölüm B.2'de tanıtılan Managed Identity, Key Vault erişiminde de Zero Trust'ın hedef durumudur: bir uygulamanın Key Vault'tan sır okuması için kendisinin bir client secret saklamasına gerek yoktur; Managed Identity ile kimlik doğrulayıp doğrudan Key Vault'a erişir. **Kural: "Bir sırrı okumak için başka bir sır saklamak zorunda kalıyorsanız, mimari yanlıştır."**

### Rotasyon, Süre Sonu ve İzleme

- **Otomatik rotasyon:** Key Vault, entegre Azure servisleri (örn. Azure SQL bağlantı dizeleri) için otomatik sır rotasyonunu destekler; mümkün olan her yerde manuel rotasyon süreçleri otomatikleştirilmelidir.
- **Süre sonu (expiration) zorunluluğu:** Her secret/certificate için bir son kullanma tarihi tanımlanmalı; süresiz (never expire) sırlar, Bölüm B.1'deki servis hesabı yayılması sorununun anahtar/sır karşılığıdır.
- **Microsoft Defender for Key Vault:** Anormal erişim desenlerini (alışılmadık konumdan/uygulamadan gelen toplu sır okuma denemesi gibi) tespit eder ve Sentinel'e sinyal besler.
- **Ağ izolasyonu:** Key Vault'a erişim, mümkün olduğunda genel internetten değil, Private Endpoint üzerinden ve güvenlik duvarı kurallarıyla (Bölüm 7-8'deki NGFW segmentasyon mantığıyla uyumlu şekilde) sınırlandırılmalıdır.

### Kod Deposu Tarafında Sır Sızıntısı Önleme

GitHub/Azure DevOps gibi kod depolarında **secret scanning** (gizli anahtar tarama) özellikleri etkinleştirilmeli; bir geliştirici yanlışlıkla bir API anahtarını koda commit ettiğinde, bu otomatik olarak tespit edilip ilgili sırrın Key Vault'ta acilen rotate edilmesi süreci tetiklenmelidir. Bu kontrol, "shift-left" güvenlik yaklaşımının (güvenliği geliştirme sürecinin en başına taşıma) kimlik/sır güvenliğindeki doğrudan karşılığıdır.

---

## Faz 1 (Adım 1) Tamamlanma Kriterleri

- Tüm insan kullanıcılar için MFA zorunlu hale getirilmiş, eski kimlik doğrulama protokolleri kapatılmış olmalı.
- Servis hesabı/service principal envanteri çıkarılmış; her birine bir sahip atanmış, 90+ gün kullanılmayanlar devre dışı bırakılmış olmalı.
- Yeni geliştirilen/güncellenen iş yüklerinde client secret yerine Managed Identity veya Workload Identity Federation kullanımı standart hale gelmiş olmalı.
- En az bir bulut ayrıcalık zincirleme (attack path) analizi yapılmış ve kritik yollar (örn. düşük ayrıcalıklı kimlikten Global Admin'e uzanan yol) kapatılmış olmalı.
- Tier 0 varlıkları tanımlanmış ve bu varlıklara uzanan tüm ilişki yolları (doğrudan/dolaylı) haritalanmış olmalı.
- API izinlerinde "Application" tipi geniş izinler gözden geçirilmiş, mümkün olanlar kaynak-seviyesi kısıtlamalarla (örn. Application Access Policy) daraltılmış olmalı; kullanıcı consent'i kısıtlanmış olmalı.
- Kod içine gömülü sır taraması (secret scanning) etkinleştirilmiş, kritik sırlar Key Vault'a taşınmış ve süre sonu/rotasyon politikası tanımlanmış olmalı.

## Sonraki Bölüm

Bölüm 4: **Koşullu Erişim ve Risk Tabanlı Kimlik Doğrulama** — burada kurulan insan ve insan dışı kimlik temelinin, dinamik ve risk tabanlı erişim kararlarına nasıl dönüştürüleceğini ele alacağız.
