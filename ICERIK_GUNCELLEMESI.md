# Block Rise – İçerik ve Denge Güncellemesi

## 1. Enerji sistemi kaldırıldı
- `Energy.cs`, enerji ikonu, enerji çipi/popup'ı, "reklam izle → enerji" akışı ve ilgili çeviri anahtarları silindi.
- Classic ve Adventure artık sınırsız ve ücretsiz oynanıyor; tekrar denemek serbest.
- Reklam kaldırma satın alımı artık yalnızca reklamları kaldırıyor.

## 2. Adventure: 1000 bölüm + piksel resimler
- **Sayfa:** Piramit haritasının yerinde Block Blast tarzı bir görünüm var. Her 20 bölüm bir piksel resim. Tamamlanan her bölüm resmin bir parçasını blok blok, ses eşliğinde açıyor. Resim bitince tebrik ekranı çıkıyor ve sonraki macera açılıyor.
- **Açılma payları:** Zor bölümler (9, 15, 18), ara boss (10), boss (19) ve resmi tamamlayan final (20) daha büyük parça açıyor.
- **50 resim:** Çilek, kedi, mantar … dağ ve kupa (`Resources/BlockRiseAdventures.json`). İsimler 12 dilde.
- **Bölüm 251-1000 (750 yeni bölüm):** 14 bölüm tipiyle tasarlandı. Her maceranın bir "spot ışığı" mekaniği, tanıtım bölümü, remix bölümleri, ödül ve nefes bölümleri, ara boss, boss ve resmi tamamlayan bir final bölümü var. Bölüm tipleri:
  - harvest, palette, purist, excavation, flood
  - demolition, storm, linebreaker, cascade, sprint
  - confetti, blueprint, arsenal, marathon
- **Yeni mekanikler sırayla açılıyor.** Hiçbir yeni ayar kendi tanıtım bölümünden önce görünmüyor:
  - 262: satır/sütun bombası
  - 282: yüksek yığın
  - 302: zincir
  - 322: tek boşluklu sel
  - 362: sabit tohum
  - 422: özel bloksuz
  - 462: hepsi birden
- **Simülasyonla ayarlandı.** Her bölüm oyunun gerçek tahta kodu ve üç bot tipiyle (uzman, ortalama, acemi) binlerce kez oynatıldı. Hedef miktarlar, ortalama oyuncunun ilk deneme kazanma oranına göre ayarlandı. Ek korumalar:
  - Acemi oyuncu kazanabilmeli.
  - Uzman, ortalama oyuncudan iyi olmalı (tohum tuzağı olmamalı).
  - Kayıpların çoğu "kıl payı" olmalı.
  - Yardımla en geç 3. denemede büyük ihtimalle geçilmeli.
- **Kurallar:** Blok hedefi hiçbir zaman bağlayıcı hedef değil (eski bölümlerdeki ani zorluk sıçramalarının sebebiydi). Yıldız eşikleri kazanılan hamle dağılımından çıkarıldı; artık her galibiyet 3 yıldız değil.
- Boss bölümlerinde sabit tohum var: aynı başlangıç, öğrenilebilir.

## 3. Classic modun zorluk algoritması (yeni yönetmen)
- **Kademeler:** Yaklaşık her 12-20 turda bir STAGE açılıyor. Taban zorluk zirveye kadar artıyor, zirveden sonra "uzatma" ile yavaşça sertleşmeye devam ediyor. Her oyun eninde sonunda bitiyor, nerede biteceğini beceri belirliyor.
- **Dalga:** Her kademe üç aşamada akıyor: birikim → zirve (daha sık özel blok, daha sert sıralar) → rahatlama (yardımlı, zincir dostu sıralar, gerekirse bir yükselme atlanıyor). Büyük temizlemeler erken rahatlama kazandırıyor.
- **Adalet:**
  - Tahta yüksekken oyuncu patlatamıyorsa bir can simidi sırası geliyor.
  - Yeni oyunculara gizli kurtarma veriliyor.
  - Ölü tahta koruması var.
  - Açılışta 4 renk kullanılıyor.
- **Simülasyon sonucu (oyuncu başına 12 oyunluk kariyerin 4-12. oyunları):**

  | Oyuncu | Tur (medyan) | Tur (p90) |
  |---|---|---|
  | Uzman | ~142 | ~273 (eskiden ~700'e kadar uzuyordu) |
  | Ortalama | ~113 | ~231 |
  | Acemi | ~94 | ~182 |

  Oyunların kabaca %15-30'u tahtanın tehlikeli bölgede olduğu gergin turlar, %18-25'i rahat turlar.

## 4. Hata düzeltmesi
- Hamle kalmayınca yapılan karıştırma ve tek zorunlu yükselmeden sonra oyunun kilitlenebildiği bir durum vardı. Artık hamle açılana ya da tahta tavana ulaşana kadar sıra yükseliyor.

## Notlar
- Bölüm 1-250'nin oynanışı değişmedi. Simülasyonda bazı eski bölümlerde (225, 233, 238, 241, 244, 249) blok hedefinden kaynaklı ani zorluk sıçramaları görüldü; ileride elden geçirilebilir.
- Simülasyon araçları ve raporlar depoya dahil değil.

## Simülasyon raporu (1000 bölüm, bölüm başına 300 oyun; deneme sayıları uyarlanabilir yardım açıkken)

| Bölüm | Uzman | Ortalama | Acemi | Ort. deneme | >3 deneme | 3 yıldız (kazananlar) |
|---|---|---|---|---|---|---|
| 1-100 | 73% | 69% | 59% | 1.51 | 6% | 78% |
| 101-200 | 47% | 43% | 33% | 2.25 | 22% | 84% |
| 201-300 | 47% | 43% | 33% | 2.27 | 21% | 57% |
| 301-400 | 54% | 48% | 38% | 2.09 | 16% | 29% |
| 401-500 | 51% | 47% | 36% | 2.13 | 17% | 29% |
| 501-600 | 52% | 46% | 37% | 2.11 | 15% | 27% |
| 601-700 | 51% | 46% | 37% | 2.14 | 16% | 29% |
| 701-800 | 51% | 46% | 36% | 2.17 | 17% | 28% |
| 801-900 | 51% | 46% | 36% | 2.18 | 17% | 29% |
| 901-1000 | 48% | 45% | 35% | 2.25 | 19% | 27% |

- Kilitlenen oyun: 0 (toplam 900.000+ oyun).
- Yeni bölümlerde zorluk perde perde hafifçe artıyor; boss bölümleri %24-35, ödül/final bölümleri %60-80 ilk deneme.
- Eski bölümlerde (1-250) her galibiyet neredeyse 3 yıldızdı. Yeni bölümlerde 3 yıldız, galibiyetlerin yaklaşık %28'inde geliyor.
