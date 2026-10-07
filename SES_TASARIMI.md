# Block Rise — ses tasarımı v2 (Tetris Effect'ten ilham)

23 efekt sesinin tamamı yeniden tasarlandı. Dosya adları aynı kaldı, bu yüzden oyunda ayar değiştirmeye gerek yok.
Kaynak betikler: `BlockRise/SesKaynagi~/kit.py` (sentez araçları) ve `make.py` (sesler). Yeniden üretmek için:
`python make.py` komutunu çalıştırın. Çıktı doğrudan `Resources/BlockRiseSounds` klasörüne yazılır.

## İlkeler
- **Tek tonalite:** Her ses D majör pentatonik dizide (D E F♯ A B). Aynı anda çalan sesler birbiriyle çatışmaz.
- **Her hareket bir nota:** Sürükleme tıkları D5'e akortlu. Blok her hücre kaydığında oyun dizide bir basamak çalar: sağa kaydırınca perde yükselir, sola kaydırınca alçalır (`SoundManager.PlayScaleNote`).
- **Ağırlık:** Yerleştirme ve oturma seslerinde derin bir sub-bas "thoom" ve yumuşak bir FM akoru var.
- **Ödül hiyerarşisi:** Sık çalan sesler kısa, küçük odada. Nadir ödüller (zincir, bomba, kusursuz, kazanma) büyük salonda, shimmer yankı ve ping-pong delay ile.
- **Ses yükseklikleri v1 ile aynı hedeflerde:** Oyundaki ses dengesi bozulmaz.

## Paleti
FM kalimba ve cam çanları, supersaw pad ve "pluck" akorlar, formant koro, sub-bas süzülmesi, hava/gürültü süpürmeleri,
oktav-üstü shimmer salon yankısı, 120 BPM noktalı on altılık ping-pong delay.

## Kod değişiklikleri
- `SoundManager.PlayScaleNote`: Sürükleme notasını dizide çalar. Perde rastgele kaydırılmaz, yalnızca tını varyasyonu ve ses düzeyi değişir.
- `SoundManager.PentatonicSemitones` artık aşağı basamakları da hesaplıyor (−1 = B, −2 = A …).
- `GameSettings.organicPitchJitter` varsayılanı 0.7'den 0.12'ye indirildi. Akortlu yerleştirme akorları detone olmasın diye.
  **Not:** Projede `Resources/BlockRiseSettings.asset` dosyası varsa oradaki değer geçerli olur. O dosyada bu değeri elle 0.1–0.15 civarına çekin.
- `pop2` artık D köklü. Eskisi A köklüydü ve zincirde tizleştikçe diziden çıkıyordu.
