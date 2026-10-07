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

## Açılıştaki kasma (2. tur)
Telefonda oyun ilk açıldığında menünün ilk birkaç saniye takılmasının nedenleri ve çözümleri:

| Neden | Etki | Çözüm |
|---|---|---|
| 161 blok karosu ana thread'de, kare başına 2 tane üretiliyordu | Açılıştan sonraki ~80 kare boyunca her karede birkaç ms hesap ve toplam ~22 MB geçici dizi. Bu diziler GC duraklamalarına yol açıyordu. | Piksel hesabı artık **arka plan thread'inde**, yeniden kullanılan 6 tamponla yapılıyor. Ana thread kare başına en fazla ~1.5 ms ile yalnızca dokuları yüklüyor. Hesap kodu thread-güvenli hâle getirildi (`TileShape`). Oyun sırasında eksik bir karo istenirse tek seferlik tamponla üretiliyor, çöp oluşmuyor. WebGL'de thread olmadığı için orada zaman bütçeli eski yöntem kullanılıyor. |
| Sesler açılışta ana thread'de çözülüyordu (`preloadAudioData` açık, `loadInBackground` kapalı) | 23 Vorbis sesi PCM'e açılırken her biri bir karede 10–40 ms sürebiliyordu. 1. turda eklenen ses ön yüklemesi bu ayar yüzünden açılışı ağırlaştırmış olabilir. | Tüm seslerde **Load In Background** açıldı. Çözme artık arka planda yapılıyor. |
| Android titreşim servisi ilk titreşimde hazırlanıyordu (JNI) | İlk dokunuşta veya ilk blok sürüklemesinde 10–30 ms takılma. | `Haptics.Prewarm()` ile açılışta hazırlanıyor. |
| Shader'lar ilk çizimde GPU sürücüsünde derleniyordu | İlk görünen sprite/UI/yazıda takılma. | Açılışta `Shader.WarmupAllShaders()` çağrılıyor. |

Proje ayarlarında ayrıca şunları öneriyorum:
- **Player → Other Settings → Use incremental GC:** açık.
- **Optimized Frame Pacing:** açık.
- Gerçek ölçüm için Development Build + Autoconnect Profiler ile telefonda ilk 5 saniyeyi kaydedin.

## Yükleme ekranı (3. tur)
Açılıştaki tüm ağır hazırlıklar artık bir yükleme ekranının arkasında yapılıyor. Menü açıldığında hiçbir şey ilk kez hazırlanmıyor.

- **Ne görünüyor:** Oyunun canlı arka planı ve yıldız tozu üstünde "BLOCK RISE" logosu, oyunun blok renklerinde ilerledikçe yanan 5 blok ve ince bir ilerleme çubuğu. Bitince ekran yumuşakça kayboluyor (`UI/LoadingScreen.cs`).
- **Sırası (`GameRoot.Boot`):**
  1. Arka plan ve yükleme ekranı (ilk kare). Sesler ve blok karoları hemen arka planda hazırlanmaya başlıyor.
  2. Efektler, tahta, oyun kontrolcüsü ve tüm arayüz ekranları.
  3. Bölüm haritası önceden kuruluyor. "Levels" ekranı ilk açıldığında takılmıyor.
  4. 161 blok karosunun ve 23 sesin bitmesi bekleniyor (güvenlik sınırı en fazla 15 sn).
  5. Font dokusu: arayüzdeki tüm yazıların karakterleri önceden işleniyor.
  6. Titreşim servisi, shader'lar ve bir kez çöp toplama (GC).
  7. Menü ya da yarım kalan oyun açılıyor, yükleme ekranı kayboluyor.
- Her adımdan sonra bir kare bekleniyor, böylece yükleme ekranı donmuyor ve ilerliyor.
