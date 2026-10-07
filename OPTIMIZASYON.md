# Block Rise — performans ve boyut optimizasyonu

`BlockRise.zip` içindeki proje tarandı ve güncellendi. Açılmış boyut 9.8 MB'tan 7.3 MB'a, zip 5.98 MB'tan 5.29 MB'a indi.

## Boyut
| Dosya | Önce | Sonra | Ne yapıldı |
|---|---|---|---|
| `Resources/BlockRiseSounds/*.wav` (23) | 7.3 MB | 5.0 MB | Duyulmayan sessiz kuyruk kırpıldı (tepe değerinin −60 dB altı + 20 ms pay). Duyulan ses değişmedi. |
| `Resources/BlockRiseFont.ttf` | 156 KB | 30 KB | Kullanılmayan Devanagari glifleri çıkarıldı. Loc ve koddaki tüm karakterlerin hâlâ fontta olduğu doğrulandı. |
| `Art/icon.png` | 1.8 MB | 1.25 MB | 1254 → 1024 px (mağazaların istediği en büyük boyut). |
| `Resources/BlockRiseSprites/icon_lightning1.png` | 5 KB | silindi | Kodda hiçbir yerden kullanılmıyordu (Resources'taki her dosya build'e girer). |

Ses kırpması RAM'i de düşürür: sesler "Decompress On Load" ile açıldığı için bellekteki PCM boyutu yaklaşık %30 azalır.

## Performans
- **Kodla üretilen dokular artık RAM'de kopya tutmuyor** (`Texture2D.Apply(false, true)`): 161 blok karosu (gövde + parlama), UI blok/cam sprite'ları, yuvarlak köşeli sprite'lar ve yıldız tozu atlası. Varsayılan 128 px karo çözünürlüğünde yaklaşık **20+ MB RAM** tasarrufu. Bu dokuların hepsi FullRect sprite ya da materyal olarak kullanılıyor, pikselleri sonradan okunmuyor.
- **Karo üretimi `Color32` ile yapılıyor** (`SetPixels32`): açılıştaki karo hazırlığında üretilen çöp 4 kat azaldı, GC takılması azaldı.
- **Ses ön yükleme**: `preloadAudioData` açıldı. `SoundManager` tüm efekt seslerini açılışta, karelere yayarak `Resources.LoadAsync` ile yüklüyor. Önceden her ses ilk çaldığı karede diskten okunup çözülüyordu (ilk patlama, zincir ya da bomba anında takılma oluyordu).
- **`BlockView.Update` yalnızca özel bloklarda çalışıyor**: sıradan bloklarda bileşen kapalı (yalnızca build'de; editörde karo tazelemesi için açık kalıyor).
- Ses import ayarında `3D` kapatıldı (2D efekt sesi).

## Zaten iyi durumda olanlar (değiştirilmedi)
Blok havuzlama, hareketli UI süsleri için ayrı Canvas, karoların karelere yayılarak ön üretimi, `PressScale` erken çıkışı, tek mesh/tek draw call yıldız tozu ve parçacıklar, ayırmasız `BoardSim`, hedef kare hızı ayarı.

## Proje ayarlarında önerilenler (zip'te ProjectSettings yok)
- Scripting Backend: **IL2CPP**, Managed Stripping Level: **High**, yalnızca **ARM64**.
- Android: **App Bundle (AAB)**, texture sıkıştırma: **ASTC**.
- Kullanılmayan built-in paketleri kaldırın (ör. Physics, AI, Terrain, Video, VR/XR, Timeline). Oyun bunları kullanmıyor.
- `BlockRiseLevels.json` düzenlemeyi kolaylaştırmak için girintili bırakıldı. Küçültmek build'e sıkıştırılmış hâlde yalnızca ~10 KB kazandırır.
