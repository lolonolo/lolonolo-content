# Lolonolo İçerik Şeması

Bu repodaki JSON dosyaları, **iOS ve Android uygulamalarının store güncellemesi
beklemeden** okuduğu içeriklerdir. Bir dosyayı değiştirip commit'lediğinde,
kullanıcılar uygulamayı bir sonraki açışlarında yeni içeriği görür.

## Altın kurallar

1. **Dosya adlarını değiştirme.** Uygulamalar bu adlara göre çekiyor.
2. **Alan adlarını değiştirme veya silme.** Yeni alan eklemek serbesttir
   (eski sürümler tanımadığı alanı görmezden gelir), ama var olanı silmek
   eski sürümlerdeki ekranı bozar.
3. **`id` bir kez verilir, bir daha değişmez.** Bildirim takibi ve
   "okundu" durumu bu id'ye bağlı. Değiştirirsen kayıt yeniymiş gibi
   davranır ve kullanıcıya tekrar bildirim gider.
4. **Tarih formatı her zaman `YYYY-MM-DD`.** `05.12.2026` değil,
   `2026-12-05`.
5. Türkçe karakter serbest. Dosyalar UTF-8.

---

## takvim.json — Resmî akademik takvim

Uygulamadaki Sınav Takvimi ekranında, kullanıcının kendi eklediği kayıtların
yanında ayrı bir katman olarak görünür. Kullanıcı bu kayıtları **silemez**,
sadece görüntüler.

### Kayıt alanları

| Alan | Zorunlu | Açıklama |
|---|---|---|
| `id` | ✅ | Benzersiz. Örn: `auzef-2026-guz-vize`. Bir kez verilir, değişmez. |
| `kurum` | ✅ | `AUZEF` / `ANADOLU` / `ATA` / `SEGEM` |
| `baslik` | ✅ | Kısa ve net. Örn: "Güz Dönemi Ara Sınavı (Vize)" |
| `aciklama` | – | Uzun olabilir. Boş bırakılabilir (`""`). |
| `baslangic` | ✅ | `YYYY-MM-DD` |
| `bitis` | – | Çok günlü etkinlikler için. Tek günlükse yazma. |
| `durum` | ✅ | `kesinlesti` / `tahmini` |
| `tip` | ✅ | `VIZE` / `FINAL` / `BUTUNLEME` / `UCDERS` / `BASVURU` / `DUYURU` |
| `link` | – | lolonolo.com'daki ilgili sayfa. Boşsa buton görünmez. |

### `durum` alanı — en kritik alan

Resmî duyuru çıkmadan önce tahmini tarih paylaşmak faydalı, ama
**etiketlenmemiş yanlış tarih güven kaybettirir.**

- `tahmini` → uygulamada sarı "Tahmini" etiketi görünür, kullanıcı bunun
  kesinleşmediğini bilir.
- `kesinlesti` → etiket yok, tarih resmî kabul edilir.

Resmî takvim açıklandığında `tahmini` → `kesinlesti` yapmayı unutma.
Tarih de değiştiyse `baslangic`'ı güncelle — kullanıcılara otomatik
"tarih güncellendi" bildirimi gider.

---

## duyurular.json — LoloHaber akışı

Şu ana kadar duyurular tamamen push bildirime bağlıydı: bildirim izni
vermeyen ya da teslimatı kaçıran kullanıcı hiçbir şey görmüyordu. Bu dosya
o sorunu çözer — uygulama her açılışta bunu çeker, push'tan bağımsız
**garantili** bir duyuru akışı oluşturur. Push artık sadece "yeni bir şey
var" tetikleyicisi; asıl içerik buradan gelir.

### Kayıt alanları

| Alan | Zorunlu | Açıklama |
|---|---|---|
| `id` | ✅ | Benzersiz. Bir kez verilir, değişmez (değiştirirsen kullanıcıya tekrar "yeni" gibi görünür). |
| `baslik` | ✅ | Kısa başlık. |
| `icerik` | ✅ | Uzun olabilir — push payload sınırı yok artık. |
| `kategori` | ✅ | `DUYURU` / `SINAV` / `ICERIK` / `ACIL` |
| `tarih` | ✅ | `YYYY-MM-DD` — listede sıralama ve gösterim için. |
| `baslangic` | – | Bu tarihten önce kullanıcıya gösterilmez. Boşsa hemen görünür. |
| `bitis` | – | Bu tarihten sonra otomatik kaybolur. Boşsa süresiz görünür. |
| `hedef` | – | `quiz:ID` (uygulama içi, ileride) / `web:URL` (bağlantı butonu) / boş |
| `bolum` | – | Opsiyonel filtre etiketi (şu an UI'da kullanılmıyor, ileride eklenebilir). |

### `baslangic` / `bitis` ne işe yarar

Bir kampanya duyurusunu "5 gün sonra otomatik kaybolsun" diye ayarlamak
istersen `bitis`'i doldur — o tarihten sonra dosyadan silmene gerek kalmaz,
uygulama kendisi gizler. `baslangic` ise ileri tarihli bir duyuruyu
şimdiden hazırlayıp push'lamana izin verir, ama o tarihe kadar kimse görmez.

### `hedef`

- `web:https://lolonolo.com/...` → duyuru kartında "Devamını Oku" butonu
  bu linke gider.
- `quiz:SORU_ID` → şu an sadece taşınır, uygulama henüz buna göre
  yönlendirme yapmıyor (gelecekte quiz ekranına derin bağlantı için ayrılmış).
- Boş → buton görünmez, sadece metin.

### `kurum` alanı

Kullanıcı uygulamada kendi kurumunu seçiyor. AUZEF öğrencisine `ATA`
kayıtları gösterilmez. Kurumu doğru yazmak önemli — yanlış kurum, kaydın
kimseye görünmemesi demek.

---

## config.json — Kontrol paneli

Bu dosya içerik taşımaz, **uygulamanın davranışını** kontrol eder.
Acil durum sigortasıdır.

### `ekranlar` — kill switch

Bir ekran bozulursa (örn. sinav.lolonolo.com çöktü, WebView hata veriyor)
o ekranı uzaktan kapatabilirsin:

```json
"sinav_merkezi": { "acik": false, "mesaj": "Sınav merkezi bakımda, yakında döneceğiz." }
```

Kullanıcı o karta dokunduğunda hata ekranı yerine bu mesajı görür.
Sorun çözülünce `true` yap. **Store onayı beklemene gerek kalmaz.**

### `surum_uyarisi` — güncelleme çağrısı

| Alan | Açıklama |
|---|---|
| `aktif` | `true` ise uyarı gösterilir |
| `zorunlu` | `true` ise kullanıcı kapatamaz, güncellemeden devam edemez |
| `min_android_build` | Bu **build numarasının altındaki** Android sürümlerine gösterilir |
| `min_ios_build` | Bu build numarasının altındaki iOS sürümlerine gösterilir |

> **Dikkat:** iOS ve Android'in numaraları birbirinden bağımsız.
> Android şu an `22`, iOS şu an `4`. Ayrı ayrı yaz.
> `0` yazarsan o platformda kimseye gösterilmez.

`zorunlu: true` güçlü bir silahtır — yanlış numarayla herkesi kilitleyebilirsin.
Önce `zorunlu: false` ile dene.

---

## Değişiklik yaptıktan sonra

1. Commit'le ve push'la.
2. GitHub Action otomatik olarak:
   - JSON geçerli mi kontrol eder (bozuksa **push reddedilir**)
   - jsDelivr CDN önbelleğini temizler
3. Kullanıcılar uygulamayı bir sonraki açışlarında yeni içeriği görür.

Önbellek temizlenmezse değişiklik **12 saat** görünmez — bu yüzden
Action'ı devre dışı bırakma.
