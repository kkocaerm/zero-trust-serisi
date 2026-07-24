---
layout: default
title: "Microsoft Purview ile Veri Koruma"
parent: "Faz 3: Uygulama ve Veri Güvenliği"
nav_order: 2
---

# Bölüm 10 (Faz 3): Veri Sınıflandırma ve Koruma — Microsoft Purview ile Bilgi Koruması ve DLP

## Yönetici Özeti

Zero Trust'ın nihai koruma hedefi her zaman veridir; kimlik, cihaz ve ağ kontrolleri veriye giden yolu korur, ancak veri kurum dışına çıktığında (e-posta eki, USB, kişisel bulut hesabı) bu kontrollerin hiçbiri devrede değildir. Microsoft Purview, veriyi kendisiyle birlikte taşınan bir koruma katmanına (etiketleme ve şifreleme) bağlayarak bu boşluğu kapatır: veri nerede olursa olsun, kimliğe bağlı izin politikası onunla birlikte hareket eder.

Yönetim için bu fazın gerekçesi, düzenleyici uyum (KVKK, GDPR, sektörel mevzuat) ve fikri mülkiyet/iş sırrı korumasıdır; veri sınıflandırması olmadan DLP veya CASB politikaları "her şeyi" veya "hiçbir şeyi" koruyan kaba araçlara dönüşür. Bölüm 2.5'teki tabloda veri sütununün karşılığı, burada işlenen Purview DLP, Retention ve Sensitivity Label Policy nesneleridir.

## Hassasiyet Etiketleri (Sensitivity Labels)

Purview Information Protection'ın temel yapı taşı hassasiyet etiketleridir (örn. Genel, Dahili, Gizli, Çok Gizli). Bir etiket uygulandığında üç şey gerçekleşir:

1. **Görsel işaretleme:** Belgeye/e-postaya başlık, altbilgi veya filigran eklenir.
2. **Erişim kısıtlaması (şifreleme):** Etiket, kimlerin belgeyi açabileceğini, düzenleyebileceğini, yazdırabileceğini veya yönlendirebileceğini tanımlayan haklarla şifrelenir (Azure Rights Management). Bu şifreleme, belge OneDrive'dan indirilse, e-postayla gönderilse veya USB ile kopyalansa bile geçerliliğini korur.
3. **Otomatik politika tetikleme:** Etiket, DLP kurallarının ve CASB oturum politikalarının (Bölüm 9) hangi içerik için tetikleneceğini belirler.

Etiketler, kullanıcı tarafından manuel uygulanabileceği gibi, içerik analizi (örn. kimlik numarası deseni, kredi kartı deseni tespiti) ile **otomatik** veya kullanıcıya öneri şeklinde de uygulanabilir; olgun bir Zero Trust uygulamasında otomatik/önerilen etiketleme oranı yüksek olmalıdır, çünkü manuel etiketlemeye bağımlılık tutarsızlık yaratır.

## Veri Kaybı Önleme (DLP) Politikaları

DLP politikaları, hassasiyet etiketleri ve/veya doğrudan içerik analizi temelinde, verinin kanallar arası hareketini denetler:

- **Uç nokta DLP:** USB'ye kopyalama, yazdırma, panoya kopyalama gibi yerel eylemleri kısıtlar (Defender for Endpoint ile entegre).
- **E-posta DLP:** Hassas içerik içeren e-postaların dış alıcılara gönderilmesini engeller veya şifreleme zorunlu kılar (Exchange Online).
- **SharePoint/OneDrive DLP:** Hassas belgelerin harici paylaşım bağlantılarıyla paylaşılmasını sınırlar.
- **Bulut uygulaması DLP:** Bölüm 9'da ele alınan CASB session policy'leriyle birleşerek, üçüncü taraf SaaS uygulamalarına yüklenen içeriği de kapsar.

## Insider Risk Management: İçeriden Gelen Tehdit

Dış saldırganlar kadar, kasıtlı veya kasıtsız içeriden veri sızıntısı da Zero Trust kapsamındadır. Purview Insider Risk Management, işten ayrılma sürecindeki bir çalışanın anormal toplu indirme davranışı, yetkisiz veri sızdırma girişimleri gibi senaryoları, HR sinyalleri (örn. istifa tarihi) ile davranışsal sinyalleri birleştirerek tespit eder ve risk skoruna göre soruşturma öncelik sırası oluşturur.

## Veri Yaşam Döngüsü ve Kayıt Yönetimi

Zero Trust'ın veri sütunu sadece erişim kontrolüyle sınırlı değildir; verinin ne kadar süre tutulacağı (retention) ve ne zaman güvenli şekilde silineceği (disposition) de saldırı yüzeyini etkiler — tutulmayan veri çalınamaz. Purview Data Lifecycle Management, etiket bazlı saklama politikalarıyla bu döngüyü otomatikleştirir.

## Yapılandırılmış Veri ve Veritabanı Seviyesi Koruma

Purview'in kapsamı dosya/e-posta gibi yapılandırılmamış veriyle sınırlı değildir; Microsoft Purview Data Map ve ilgili yönetişim araçları, kurumsal veritabanları ve veri ambarlarındaki hassas alanların (örn. müşteri kimlik numarası kolonu) keşfini ve sınıflandırılmasını da kapsar — bu, özellikle veri analitiği/BI ekiplerinin geniş veri setlerine erişiminin Zero Trust prensipleriyle uyumlu kalmasını sağlar.

## Faz 3 (Adım 2) Tamamlanma Kriterleri

- Kurum genelinde standart bir hassasiyet etiketi taksonomisi tanımlanmış ve yayınlanmış olmalı.
- En azından en kritik veri kategorileri için otomatik/önerilen etiketleme aktif olmalı.
- Uç nokta, e-posta ve SharePoint/OneDrive kanallarında DLP politikaları devrede olmalı.
- Insider Risk Management, en az işten çıkış senaryosu için yapılandırılmış olmalı.

## Sonraki Bölüm

Bölüm 11: **Görünürlük ve Otomasyon — Microsoft Sentinel ile SIEM/SOAR Entegrasyonu** — şimdiye kadar kurulan tüm sinyalleri (kimlik, cihaz, ağ, CASB, veri) tek bir merkezi görünürlük ve otomasyon katmanında nasıl birleştireceğimizi ele alacağız.
