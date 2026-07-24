---
layout: default
title: "Sürekli Uyarlanabilir Güven"
parent: "Faz 4: Optimal Olgunluk"
nav_order: 2
---

# Bölüm 13 (Faz 4): Sürekli Uyarlanabilir Güven — UEBA, AI/ML Tabanlı Tehdit Tespiti ve Otomatik Müdahale

## Yönetici Özeti

Önceki fazlarda kurulan tüm kontroller (Conditional Access, NAC, NGFW, CASB, DLP) büyük ölçüde **politika tabanlı** çalışır: önceden tanımlanmış kurallar, önceden tanımlanmış koşullara göre uygulanır. Optimal Zero Trust seviyesinin ayırt edici özelliği, bu statik kuralların yerini **sürekli, dinamik risk skorlamasına** bırakmasıdır — Gartner'ın "Continuous Adaptive Risk and Trust Assessment (CARTA)" kavramıyla da örtüşen bu yaklaşımda, erişim kararı oturum açma anında verilip kapanmaz; oturum boyunca, her saniye yeniden değerlendirilir.

Yönetim için bu fazın değeri, gelişmiş/hedefli saldırıların (APT) çoğunlukla statik kuralları atlatacak şekilde tasarlandığı gerçeğidir; sürekli davranışsal izleme, bu tür saldırıları "kural ihlali" değil "normalden sapma" olarak yakalar.

## Continuous Access Evaluation (CAE)

Microsoft Entra ID'nin CAE özelliği, bu sürekli değerlendirmenin somut bir örneğidir: geleneksel OAuth token'ları belirli bir süre (örn. 1 saat) geçerlidir ve bu süre içinde token sahibine ne olursa olsun erişim devam eder. CAE ile, bir kullanıcının hesabı devre dışı bırakıldığında, şifresi değiştirildiğinde veya risk skoru kritik seviyeye çıktığında, aktif token **gerçek zamanlı** olarak (dakikalar içinde, token süresinin sonunu beklemeden) geçersiz kılınır. Bu, "erişim kararı bir kez verilir, oturum boyunca geçerlidir" varsayımını ortadan kaldırır.

## UEBA'dan Risk Skoruna: Davranışsal Taban Çizgisi

Bölüm 11'de tanıttığımız Sentinel UEBA motoru, Faz 4'te artık sadece bir tespit aracı değil, doğrudan erişim kararlarını besleyen bir girdi haline gelir. Her kullanıcı/cihaz/servis hesabı için oluşturulan davranışsal taban çizgisi (tipik oturum açma saatleri, erişilen kaynaklar, veri hacmi, coğrafi konum) sürekli güncellenir ve bu taban çizgisinden sapma, anlık bir risk skoruna dönüşür. Bu skor, Conditional Access, NGFW kullanıcı politikaları ve CASB oturum politikaları tarafından ortak bir sinyal olarak tüketilir.

## AI/ML Tabanlı Anomali Tespiti: Sinyal Füzyonu

Optimal seviyede, tek bir kaynaktan gelen anomali (örn. sadece "alışılmadık konum") tek başına yeterli bir tetikleyici değildir; asıl güç, çoklu kaynaktan gelen zayıf sinyallerin birleştirilmesinden (sensor fusion) gelir:

- Entra ID: alışılmadık oturum açma konumu/saati (zayıf sinyal).
- Defender for Endpoint: cihazda hafif şüpheli ama kesin olmayan bir süreç davranışı (zayıf sinyal).
- Cisco ISE/NGFW: aynı cihazdan, normalden farklı bir hedef sistemle iletişim girişimi (zayıf sinyal).
- CASB: aynı oturumda alışılmadık hacimde dosya erişimi (zayıf sinyal).

Bu dört zayıf sinyal tek başına eylem gerektirmeyebilir, ancak Sentinel'in korelasyon ve makine öğrenmesi motorları bunları birleştirdiğinde güçlü bir "yüksek olasılıklı ihlal" göstergesine dönüşür. Bu, Faz 0'da tanımladığımız olgunluk modelinin "Optimal" seviyesinin tam karşılığıdır.

## Otomatik Yanıt: İnsan Müdahalesinden Önce Hareket Etmek

Optimal seviyede SOAR playbook'ları (Bölüm 11), sadece bildirim göndermekle kalmaz; aşağıdaki gibi tam otomatik, geri alınabilir (reversible) aksiyonlar tanımlanır:

- Risk skoru kritik eşiği aştığında, kullanıcının tüm aktif oturumları (CAE ile) otomatik sonlandırılır ve hesap geçici olarak askıya alınır.
- Cihaz, NAC (Cisco ISE ANC) üzerinden otomatik karantina segmentine taşınır.
- SASE/NGFW politika motoru, bu kullanıcı/cihaz için tüm yeni bağlantı taleplerini otomatik reddetmeye başlar.
- Güvenlik ekibine, tüm ilgili kanıt (log, zaman çizelgesi) otomatik derlenmiş bir olay (incident) paketi olarak iletilir.

Bu otomasyonun ölçülebilir hedefi, ortalama müdahale süresini (MTTR) dakikalara, idealde saniyelere indirmektir.

## Risk Eşiklerinin Kalibrasyonu ve Yanlış Pozitif Yönetimi

Sürekli uyarlanabilir güven modelinin en büyük operasyonel riski, aşırı hassas eşiklerin meşru kullanıcıları gereksiz yere engellemesidir (yanlış pozitif/false positive). Bu nedenle Faz 4'te, risk eşikleri kademeli olarak (önce "sadece bildir", sonra "ek doğrulama iste", son olarak "otomatik engelle") devreye alınmalı ve düzenli olarak (örn. üç aylık) yanlış pozitif/negatif oranı gözden geçirilerek kalibre edilmelidir.

## Faz 4 (Adım 2) Tamamlanma Kriterleri

- CAE (Continuous Access Evaluation) tüm kritik uygulamalarda aktif olmalı.
- UEBA risk skoru, en az bir Conditional Access ve bir NGFW/CASB politikasına doğrudan girdi olarak bağlanmış olmalı.
- En az 2-3 senaryoda tam otomatik (insan onayı gerektirmeyen) yanıt playbook'u canlıya alınmış olmalı.
- Risk eşikleri için düzenli kalibrasyon süreci (yanlış pozitif/negatif takibi) operasyonel olmalı.

## Sonraki Bölüm

Bölüm 14 (son bölüm): **Optimal Seviyeye Ulaşma — Olgunluk Değerlendirmesi, Sürdürülebilirlik ve Sürekli İyileştirme** — bu dönüşüm yolculuğunu nasıl ölçüp sürdürülebilir kılacağımızı ele alacağız.
