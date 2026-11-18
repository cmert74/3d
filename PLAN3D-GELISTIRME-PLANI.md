# PLAN3D – 3B Yerleşim Planı Editörü: Geliştirme Planı

> **Bu dosya, uygulamayı kodlayacak geliştirici (veya yapay zekâ kodlayıcı) için tek referanstır.**
> Her adımı bitirdikçe `[ ]` işaretini `[x]` yap, en alttaki **İlerleme Günlüğü**'ne tarih ve kısa not ekle, sonra sıradaki adıma geç. Bir adımın tüm "Kabul Kriterleri" sağlanmadan adımı işaretleme.

---

## 0. Çalışma Kuralları (önce oku)

1. Adımları **sırayla** yap. Bir adım bitmeden sonrakine geçme.
2. Her adımın sonunda uygulama **çalışır durumda** olmalı (bozuk ara sürüm bırakma).
3. Her adım bitince bu dosyayı güncelle: kutuyu işaretle + İlerleme Günlüğü'ne yaz.
4. Hata düzeltirken tüm dosyayı yeniden yazma, yalnızca ilgili kısmı değiştir.
5. Belirsiz bir nokta varsa bu dosyadaki **Kararlar** bölümüne bak. Orada yoksa en basit, geri alınabilir seçeneği uygula ve günlüğe not düş.
6. Kapsam dışı maddeleri (Bölüm 3) **yapma**.
7. PDF desteği ekleneli.
8. Bir kutuyu yalnızca özelliği çalıştırıp denedikten sonra işaretle.

---

## 1. Amaç

Genel amaçlı, tarayıcıda çalışan bir **3B kat planı / yerleşim planı hazırlama uygulaması**. Kullanıcı:

- Duvarları ölçü girerek veya fareyle çizerek oluşturur,
- Hazır kapı ve pencere tiplerini duvarın üzerine bırakır (duvar kendiliğinden kesilir),
- Odalara ad ve ölçü etiketi verir,
- Elindeki bir taslağı (jpg/png, el çizimi, PowerPoint çıktısı) soluk bir **altlık** olarak yükleyip üzerinden çizer, sonra altlığı kaldırır,
- Planı aynı anda 2B ve 3B görür, hazır kamera açılarıyla sunum için görüntü alır,
- Projeyi dosyaya kaydeder ve tekrar açar.

Hedef kullanıcı: mimar olmayan, pratik ve hızlı sonuç isteyen biri. Arayüz sade olmalı. **Birincil platform: masaüstü tarayıcı (fare + klavye).** Tablet/telefon ileride düşünülecek, şimdilik zorunlu değil ama mimari buna engel olmamalı.

---

## 2. Teknik Kararlar

| Konu | Karar |
|---|---|
| Dağıtım | **Tek bir `index.html` dosyası** (CSS ve JS içinde). Sunucu/derleme (build) yok. Çift tıklayınca açılmalı. |
| Dil | Saf (vanilla) JavaScript, ES2020. Framework yok. |
| 3B | **three.js r128**, UMD sürümü. Tercihen dosyanın yanına `three.min.js` olarak koy (çevrimdışı çalışsın); olmazsa `https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js`. `OrbitControls` r128 UMD içinde yok, **kamera kontrolünü elle yaz** (sürükle = döndür, tekerlek = yakınlaştır, sağ tuş/orta tuş = kaydır). |
| 2B | `<canvas>` üzerinde elle çizim (ızgara, duvarlar, açıklıklar, etiketler, altlık). |
| İç birim | **Her şey santimetre (cm) olarak saklanır.** Ekranda seçilen birime çevrilir. |
| Koordinat | 2B'de x sağa, y aşağı (cm). 3B'ye çevirirken: `x3d = x`, `z3d = y`, yükseklik `y3d`. Sahneyi plan merkezine göre ortala. |
| Tarayıcı depolama | `localStorage` yalnızca ayarlar (dil, birim, son kullanılan değerler) için. **Proje verisi dosyaya kaydedilir.** |
| Dosya biçimi | `.plan3d.json` (aşağıda şema var). |
| Diller | **Türkçe (tr) ve Azerbaycanca (az)**. Tüm arayüz metinleri bir `I18N` sözlüğünde. Yeni dil eklemek sözlüğe anahtar eklemek kadar kolay olmalı. |
| Stil | Açık/koyu temayı destekle (`prefers-color-scheme`). Sade, geniş çalışma alanı. |

### Kararlar (kullanıcıdan gelen gereksinimler – değiştirme)

- Birimler seçilebilir: **mm, cm (az. dilinde "sm"), m, km** (isteğe bağlı inç/fit). Varsayılan: cm/sm. Birim adı dile göre otomatik uyar (tr: "cm", az: "sm"); kullanıcı elle de seçebilir.
- Duvar yüksekliği ve kalınlığı **sabit değil**. Her duvarın kendi değeri vardır. Panelde değiştirilebilen "varsayılan yükseklik / kalınlık" ayarı da bulunur (başlangıç: 270 cm / 20 cm).
- Kapı/pencere genişlik, yükseklik ve denizlik (zeminden yükseklik) ayarlanabilir.
- Uygulama **boş proje** ile açılır. Örnek plan eklenmez.
- Oda adları kullanıcının yazdığı gibi kalır, dil değişince çevrilmez.
- Altlık yalnızca kılavuzdur; resimden otomatik duvar çıkarma **yoktur**.

---

## 3. Kapsam Dışı (yapma)

- Resimden otomatik duvar/kapı tanıma.
- Çatı, merdiven, çok katlı bina, eğri/yay duvar (ilk sürümde sadece doğrusal duvar).
- Gerçekçi malzeme/ışık (ray tracing vb.).
- Kullanıcı hesabı, sunucu, bulut kaydı.
- Dokunmatik/mobil optimizasyon (ileriki sürüm).

---

## 4. Veri Modeli

Tüm uzunluklar cm. Proje tek bir nesnedir:

```json
{
  "version": 1,
  "settings": { "lang": "tr", "unit": "cm", "defaultWallHeight": 270, "defaultWallThickness": 20, "gridSize": 10 },
  "underlay": { "imageDataUrl": "...", "x": 0, "y": 0, "scale": 1.0, "rotation": 0, "opacity": 0.4, "visible": true, "calibrated": false },
  "walls": [
    { "id": "w1", "x1": 0, "y1": 0, "x2": 450, "y2": 0, "thickness": 20, "height": 270, "color": "#f4efe6" }
  ],
  "openings": [
    { "id": "o1", "wallId": "w1", "type": "door-single", "pos": 120, "width": 90, "height": 210, "sill": 0, "flip": false, "swing": "in" }
  ],
  "rooms": [
    { "id": "r1", "name": "Koridor", "points": [[0,0],[450,0],[450,300],[0,300]], "floor": "#d9d3c8", "showDims": true }
  ],
  "furniture": [
    { "id": "f1", "type": "table", "x": 200, "y": 150, "w": 120, "d": 80, "h": 75, "rot": 0, "color": "#a5754b" }
  ]
}
```

Notlar:
- `openings[].pos`: açıklığın **merkezinin**, duvarın başlangıç noktasından (x1,y1) itibaren uzaklığı (cm).
- Açıklık duvar sınırları dışına taşamaz; taşarsa otomatik içeri çekilir. İki açıklık üst üste binemez (en yakın boş yere kaydır veya engelle).
- Duvar silinirse üzerindeki açıklıklar da silinir. Duvar kısaltılırsa sığmayan açıklıklar uyarıyla silinir.
- Kimlikler (`id`) benzersiz ve kısa metin olsun.

---

## 5. Arayüz Düzeni (masaüstü)

```
┌────────────────────────────────────────────────────────────┐
│ Üst çubuk: Yeni | Aç | Kaydet | PNG | Geri al/Yinele | Dil | Birim │
├──────────┬──────────────────────────────┬──────────────────┤
│ Sol panel│  2B çizim alanı (canvas)     │ Sağ panel        │
│ Araçlar: │                              │ 3B önizleme      │
│ Seç      │                              │ (açı düğmeleri)  │
│ Duvar    │                              ├──────────────────┤
│ Kapılar  │                              │ Özellikler       │
│ Pencereler│                             │ (seçili nesne)   │
│ Oda      │                              │                  │
│ Mobilya  │                              │                  │
│ Altlık   │                              │                  │
└──────────┴──────────────────────────────┴──────────────────┘
Alt çubuk: imleç koordinatı, ölçek, ipuçları
```

- 2B/3B bölme genişliği sürüklenerek ayarlanabilsin; 3B'yi tam ekran yapan düğme olsun.
- Klavye: `Esc` aracı iptal eder, `Delete` seçileni siler, `Ctrl+Z / Ctrl+Y` geri al/yinele, `Ctrl+S` kaydet, boşluk basılı + sürükle = kaydır, tekerlek = yakınlaştır.

---

## 6. ADIMLAR

### ADIM 1 – İskelet, dil ve birim altyapısı
- [ ] Tek `index.html` oluştur: üst çubuk, sol panel, 2B canvas, sağ panel (3B + özellikler), alt çubuk.
- [ ] `I18N` sözlüğü (tr, az) ve `t('anahtar')` fonksiyonu; dil değişince tüm arayüz yenilensin.
- [ ] Birim sistemi: `toDisplay(cm)`, `fromDisplay(value, unit)`; birim seçici (mm, cm/sm, m, km). Sayı girişi ve etiketler bu fonksiyonlardan geçsin.
- [ ] 2B canvas: ızgara, kaydırma (pan), yakınlaştırma (zoom), imleç koordinatı alt çubukta.
- [ ] Ayarlar `localStorage`'a kaydedilsin (dil, birim).

**Kabul kriterleri:** Dil tr↔az değişince tüm metinler değişir; birim değişince alt çubuktaki koordinat doğru çevrilir; sayfa yenilenince ayarlar korunur.

### ADIM 2 – Duvar çizimi ve düzenleme (2B)
- [ ] Duvar aracı: tıkla–tıkla veya sürükleyerek çizim. Izgaraya ve diğer duvarın uç/orta noktasına yapışma (snap), `Shift` ile 0/90 derece kilidi.
- [ ] **Ölçü girişi:** çizerken klavyeden uzunluk yazıp `Enter` ile duvarı tam o uzunlukta bitir (yön, imlecin yönüdür).
- [ ] Seçme, taşıma, uç noktasını sürükleyerek uzatma/kısaltma, silme.
- [ ] Özellikler panelinde: uzunluk, kalınlık, yükseklik, renk (sayı kutuları, seçili birimde).
- [ ] Duvar köşeleri düzgün birleşsin (iki duvar uç noktada buluşunca köşede boşluk/çıkıntı kalmasın; basit çözüm: duvar uçlarını kalınlığın yarısı kadar uzatmak).
- [ ] Duvar üzerinde uzunluk etiketi gösterilsin (aç/kapa seçeneği).

**Kabul kriterleri:** 450 yazıp Enter ile tam 450 cm duvar oluşur; duvar seçilip kalınlığı değiştirilince 2B'de anında güncellenir; L şekilli iki duvarın köşesi temiz görünür.

### ADIM 3 – Altlık (taslak) içe aktarma ve ölçekleme
- [ ] "Altlık yükle": jpg/png seç (dosya seçici + sürükle-bırak). PDF'in ilk sayfası
- [ ] Altlık, duvarların **altında** soluk görünür; saydamlık kaydırıcısı, taşıma, döndürme, göster/gizle, sil.
- [ ] **Ölçek ayarı:** "Ölçeklendir" aracı → altlıkta iki nokta tıkla → gerçek mesafeyi (seçili birimde) yaz → altlık orantılı olarak o ölçeğe getirilir. `calibrated=true` olur.
- [ ] Altlığı kilitle (yanlışlıkla kaymasın) seçeneği.
- [ ] Altlık projeyle birlikte kaydedilebilsin (dataURL) veya "altlığı dosyaya ekleme" seçeneği olsun.
- [ ] Altlık gizlenince/silinince çizilen plan etkilenmesin.

**Kabul kriterleri:** 1000 px genişliğinde bir resimde 958 cm'lik bir kenar işaretlenip 958 girilince, üzerine çizilen 958 cm'lik duvar altlıktaki o kenarın üzerine tam oturur.

### ADIM 4 – Kapı ve pencere (otomatik duvar kesme)
- [ ] Sol panelde hazır tipler (küçük simgeyle):
  - Kapılar: tek kanat (90×210), çift kanat (160×210), sürgülü (180×210), açık geçit/boşluk (100×210).
  - Pencereler: standart (120×120, denizlik 90), geniş (200×140, denizlik 80), küçük/banyo (60×60, denizlik 150), kapı-pencere/balkon kapısı (90×210).
- [ ] Panelden seçip **duvarın üzerine bırak**: imleç duvara yaklaşınca açıklığın o duvara yapışmış önizlemesi görünsün; bırakınca `openings`'e eklensin.
- [ ] 3B'de duvar **gerçekten kesilir** (denizlik altı ve lento üstü duvar parçası kalır; pencerede yarı saydam cam, kapıda isteğe bağlı kapı kanadı veya sadece boşluk). Üst üste binme olmaz.
- [ ] Seçilen açıklık duvar boyunca sürüklenebilsin; özellikler panelinde genişlik, yükseklik, denizlik, ve "duvar başından uzaklık" sayısal girilebilsin. Kapı için yön çevirme (`flip`, içe/dışa açılma).
- [ ] 2B'de standart mimari semboller: kapıda kanat + açılma yayı, pencerede çift çizgi/cam çizgisi.
- [ ] İki açıklık çakışamaz; duvar sınırını aşamaz.

**Kabul kriterleri:** Pencereyi duvarın ortasına bırakınca 3B'de duvarda tam o ölçüde delik oluşur; sürükleyince delik de kayar; ikinci bir kapıyı üst üste bırakmaya çalışınca engellenir veya yakın boş yere kayar.

### ADIM 5 – Oda etiketleri ve alan hesabı
- [ ] Oda aracı: kapalı alanı tıklayarak noktalarla çiz **veya** duvarlarla çevrili bir boşluğun içine tıklayınca otomatik algıla (zorunlu değil; zor ise yalnızca elle çizim yap).
- [ ] Oda adı yazılır (serbest metin). Zemin etiketi: ad + ölçü (örn. `KORİDOR 958 × 320 sm`) ve alan (m²). Ölçü, odanın sınırlayıcı dikdörtgeninden; düzensiz odada yalnızca alan gösterilir.
- [ ] Etiket 2B'de odanın içinde, 3B'de **zemin üzerinde okunur şekilde** görünsün (mobilyanın altında kalmasın: etiket zeminin hemen üstünde ve mobilyadan bağımsız çizilsin).
- [ ] Zemin rengi/dokusu seçenekleri (ahşap, taş, fayans, beton, düz renk – basit renk/desen yeterli).
- [ ] Etiketlerin yazı boyutu ve gösterilip gizlenmesi ayarlanabilir.

**Kabul kriterleri:** 450×300 cm'lik oda çizilip "Salon" yazılınca "SALON 450 × 300 cm" ve "13,5 m²" görünür; dil/birim değişince ölçü yazısı güncellenir, oda adı değişmez.

### ADIM 6 – 3B önizleme, açılar ve çıktı
- [ ] Duvarlar, açıklıklar, zeminler, etiketler 3B'de gerçek zamanlı güncellensin (2B'de yapılan değişiklik hemen yansısın).
- [ ] Yumuşak gölge, hemisfer + yönlü ışık. Açık renkli, modern, sade görünüm.
- [ ] Hazır açı düğmeleri (yumuşak geçişli kamera): **Plan (yukarıdan), İzometrik, Perspektif, Yan görünüş, Köşe**. Plan merkezine ve boyutuna göre otomatik uzaklık.
- [ ] Serbest dolaşma: sürükle = döndür, tekerlek = yakınlaştır, sağ/orta tuş = kaydır.
- [ ] "Üst kesit" (duvarları kes) seçeneği: duvar yüksekliğini geçici olarak %40'a indir (içerisi daha rahat görünsün).
- [ ] **PNG olarak kaydet**: mevcut 3B görünümü (ve istenirse 2B planı) yüksek çözünürlükle (örn. 2×) indir. Çizim tamponunu koruma ayarı (`preserveDrawingBuffer`) veya render sonrası `toDataURL` kullan.

**Kabul kriterleri:** Açı düğmesine basınca kamera yumuşakça döner; PNG dosyası indirilir ve etiketler okunaklıdır.

### ADIM 7 – Kaydet / Aç / Geri al
- [ ] **Kaydet:** projeyi `.plan3d.json` olarak indir. **Aç:** dosyadan yükle (sürükle-bırak da çalışsın). Şema `version` kontrolü yap, hatalı dosyada anlaşılır hata mesajı ver.
- [ ] **Geri al / Yinele** (en az 50 adım): tüm değişiklikler (çizim, taşıma, özellik değişikliği, ekleme/silme) kapsansın.
- [ ] Otomatik yedek: değişiklik oldukça `localStorage`'a son durumu yaz; sayfa kazara kapanıp açılırsa "Son oturumu geri yükle?" sor.
- [ ] Kaydedilmemiş değişiklikle sayfadan çıkarken uyarı.

**Kabul kriterleri:** Kaydedilen dosya yeni sekmede açılınca birebir aynı plan görünür (altlık dahil, seçildiyse). 20 adım geri alınıp yinelenebilir.

### ADIM 8 – Mobilya kütüphanesi
- [ ] Basit parametrik parçalar: masa, sandalye, koltuk, yatak, dolap, tezgâh/mutfak reyonu, lavabo, klozet, küvet/duş, kapı önü/halı vb. (kutu ve basit geometrilerle).
- [ ] Panelden sürükleyip bırak; seçip taşı, döndür (15° adım), boyutlandır (en/boy/yükseklik), renk değiştir, sil, kopyala.
- [ ] 2B'de üstten basit simge/çizgi, 3B'de basit hacim.

**Kabul kriterleri:** Masa bırakılıp 200×100 cm yapılınca hem 2B hem 3B'de doğru boyutta görünür.

### ADIM 9 – Rahatlık ve cila
- [ ] Çoklu seçim (kutu ile), toplu silme/taşıma.
- [ ] Kopyala/yapıştır, duvarı ötele/paralel kopya (offset).
- [ ] Ölçü çizgisi aracı (iki nokta arası mesafe göster).
- [ ] Yardım ekranı: kısayollar ve kısa kullanım (tr/az).
- [ ] Performans: 200 duvar + 100 açıklıkta akıcı kalmalı (sahneyi sadece değişen kısım için yeniden kur veya toplu geometri kullan).
- [ ] Erişilebilirlik: düğmelerde `title`/`aria-label`, yeterli kontrast.
- [ ] (İleride) Dokunmatik destek için `pointer` olaylarını kullan, fare-özel kod yazmaktan kaçın.

**Kabul kriterleri:** Kısayollar çalışır; büyük projede takılma olmaz; hiçbir adımda konsolda hata yoktur.

---

## 7. Test Senaryosu (her büyük adımdan sonra elle dene)

1. Dili Azerbaycanca yap, birimi "sm" bırak; 450 sm'lik bir duvar çiz.
2. 4 duvarla 450×300'lük kapalı bir oda yap, "Salon" adını ver. Etiket ve m² doğru mu?
3. Bir duvara kapı, bir duvara pencere bırak; kaydır, boyutlandır. 3B'de delikler doğru mu?
4. Bir resim yükle, ölçekle, üzerinden en az 3 duvar çiz, altlığı sil. Plan bozulmadı mı?
5. Kaydet → sayfayı yenile → Aç. Her şey aynı mı?
6. Birimi m yap; tüm ölçüler doğru dönüştü mü? Dili Türkçeye çevir; metinler değişti mi, oda adı aynı kaldı mı?

---

## 8. İlerleme Günlüğü

> Her adımın bitiminde buraya ekle: `YYYY-AA-GG – ADIM N – ne yapıldı – bilinen eksikler`

- (henüz başlanmadı)

---

## 9. Açık Notlar / Sorular

- (Geliştirici belirsiz bir noktada burada not alsın.)
