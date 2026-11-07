# GO - Slices

# Go'da Dilimler (Slices)

## Dilim Nedir?

Dilim (Slice), Go dilinde dizilerin üzerine kurulan, esnek ve dinamik boyutlu bir veri yapısıdır. Dilim, altında yatan bir dizinin bir bölümüne referans tutar. Boyutu sabit değildir; `append` fonksiyonu ile eleman eklendikçe otomatik büyür. Go'da en sık kullanılan koleksiyon yapısıdır.

Kaynak: [Go Resmi Dokümantasyonu - Slices](https://go.dev/tour/moretypes/7)

## Söz Dizimi (Syntax)

```go
// Dilim tanımlama biçimleri

var dilim []int                    // Boş dilim tanımlama (nil)

dilim2 := []int{1, 2, 3}           // Değer atayarak tanımlama

dilim3 := make([]int, 5)           // make ile uzunluğu 5 olan dilim oluşturma

dilim4 := make([]string, 0, 10)    // Uzunluğu 0, kapasitesi 10 olan dilim

dilim5 := dizi[1:4]                // Mevcut diziden dilim alma (1, 2 ve 3. indeksler)
```

Dilimin üç bileşeni vardır: işaretçi (alttaki dizinin başlangıcı), uzunluk (`len`) ve kapasite (`cap`).

## Kullanım Alanları

- Önceden sayısı bilinmeyen veri kümelerinin saklanması (kullanıcı listesi, ürün listesi)
- Dosya ve veri akışlarını okumada tampon olarak kullanma
- Filtreleme, sıralama ve bölütleme işlemleri
- API'lerden dönen dinamik veri kümelerini tutma
- JSON verilerinin işlenmesi

## Kullanım Türleri

```go
// 1. Eleman ekleme (dilim otomatik büyür)
dilim = append(dilim, 10)

// 2. Çoklu eleman ekleme
dilim = append(dilim, 20, 30, 40)

// 3. Uzunluk ve kapasite öğrenme
fmt.Println(len(dilim), cap(dilim))

// 4. Dilimden bölüt alma
altDilim := dilim[1:3]

// 5. range ile döngü
for indeks, deger := range dilim {
    fmt.Println(indeks, deger)
}

// 6. Eleman silme (2. indeksi silme)
dilim = append(dilim[:2], dilim[3:]...)

// 7. Dilim kopyalama
kopya := make([]int, len(dilim))
copy(kopya, dilim)
```

## Örnek Proje: Alışveriş Sepeti Uygulaması

Bir e-ticaret sepetini dilim ile yöneten örnek:

```go
package main

import "fmt" // Formatlı çıktı kütüphanesi

func main() {
    // Sepetteki ürünleri tutan boş bir string dilimi oluşturma
    sepet := []string{}

    // Sepete ürün ekleme, append dilimi otomatik büyütür
    sepet = append(sepet, "Laptop", "Mouse", "Klavye")
    fmt.Println("Sepet:", sepet)

    // Dilimin uzunluğunu (eleman sayısı) yazdırma
    fmt.Println("Ürün Sayısı:", len(sepet))

    // Sepetten ürün silme (1. indeksteki Mouse ürünü)
    sepet = append(sepet[:1], sepet[2:]...)
    fmt.Println("Silme Sonrası:", sepet)

    // Sepete yeni ürün ekleme
    sepet = append(sepet, "Monitör", "Kulaklık")
    fmt.Println("Güncel Sepet:", sepet)

    // Sepetteki ürünleri numaralandırarak listeleme
    fmt.Println("Sepet Detayı:")
    for indeks, urun := range sepet {
        fmt.Printf("%d. %s\n", indeks+1, urun)
    }

    // Dilimden bölüt alma: ilk iki ürün
    oneriler := sepet[:2]
    fmt.Println("Önerilen Ürünler:", oneriler)
}
```

### Kod Çıktısı

```
Sepet: [Laptop Mouse Klavye]
Ürün Sayısı: 3
Silme Sonrası: [Laptop Klavye]
Güncel Sepet: [Laptop Klavye Monitör Kulaklık]
Sepet Detayı:
1. Laptop
2. Klavye
3. Monitör
4. Kulaklık
Önerilen Ürünler: [Laptop Klavye]
```

## En İyi Uygulamalar (Best Practices)

- Boyutu önceden tahmin edilebiliyorsa `make` ile kapasite belirleyin; bu, gereksiz bellek kopyalamayı önler.
- `append` sonucunu her zaman aynı değişkene atayın.
- Dilimi başka bir fonksiyona gönderirken paylaşımlı bellek olduğunun bilincinde olun.
- Bağımsız bir kopya gerekiyorsa `copy` veya `slices.Clone` kullanın.
- Eleman silme işlemleri için Go 1.21 ve sonrasında `slices.Delete` tercih edin.

```go
// Kapasite önceden belirlenmiş verimli dilim oluşturma
// Yaklaşık 100 kayıt bekleniyorsa
kayitlar := make([]string, 0, 100)
```

## Sık Yapılan Hatalar ve Uyarılar

> **Uyarı:** `append` sonucunu atamayı unutmayın. `append(dilim, x)` ifadesi mevcut dilimi değiştirmez; yeni (veya büyütülmüş) dilimi döndürür. Sonuç atanmazsa eklenen eleman kaybolur.

> **Uyarı:** Dilimler alttaki diziyi paylaşır. Bir dilimden alınan bölütte yapılan değişiklik, aynı diziyi referans alan diğer dilimlerde de görünür. Beklenmeyen sonuçlara yol açabilir.

> **Uyarı:** Boş tanımlı bir dilim `nil` değerindedir. `var d []int` ile `d := []int{}` aynı değildir; ilki `nil`'dir. Ancak `len` ve `append` her ikisiyle de güvenle çalışır. JSON çıktısında `nil` dilim `null`, boş dilim `[]` olarak görünür.

> **Uyarı:** Dilim bölütü alırken sınır kontrolü yapın. Uzunluğun üzerindeki indekse erişim, `len` aşımı durumunda çalışma zamanı paniğe (panic) yol açar.

Kaynak: [Go Blog - Arrays, slices and strings](https://go.dev/blog/slices-intro)

## Kütüphane Referansları

Kodda kullanılan `fmt` paketi Go standart kütüphanesine aittir. Detaylar: [fmt paketi](https://pkg.go.dev/fmt)

Dilimler için yardımcı fonksiyonlar içeren `slices` paketi. Detaylar: [slices paketi](https://pkg.go.dev/slices)

## Sonuç

Dilimler, Go'da dinamik boyutlu veri kümelerini yönetmenin standart yoludur. Dizilerden farklı olarak boyutları esnektir ve `append` ile kolayca büyütülür. Uzunluk ve kapasite kavramlarını anlamak, bellek verimli kod yazmayı sağlar. Paylaşımlı bellek yapısı nedeniyle `append` sonucunun atanması ve dilim kopyalama konularına dikkat edilmelidir. Sabit boyutlu verilerde diziler, dinamik verilerde dilimler tercih edilmelidir.
