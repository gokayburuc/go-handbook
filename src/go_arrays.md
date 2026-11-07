# GO - Arrays

## Array - Dizi Nedir?

Dizi (Array), aynı türden birden fazla değeri tek bir değişken altında saklayan, sabit boyutlu bir veri yapısıdır. Go dilinde dizinin boyutu tanımlandığı anda belirlenir ve sonradan değiştirilemez. Elemanlar ardışık bellek alanında saklanır ve indeks numarası ile erişilir. İlk elemanın indeksi `0`'dır.

Kaynak: [Go Resmi Dokümantasyonu - Arrays](https://go.dev/tour/moretypes/6)

## Söz Dizimi (Syntax)

```go
// Tek boyutlu dizi tanımlama biçimleri

var dizi [5]int                    // Boyutu belirterek tanımlama, varsayılan değerler 0

var dizi2 = [5]int{1, 2, 3}        // Değer atayarak tanımlama

dizi3 := [3]string{"a", "b", "c"} // Kısa tanımlama

dizi4 := [...]int{10, 20, 30}      // Boyutu derleyicinin hesaplamasına bırakma
```

## Kullanım Alanları

- Sabit sayıda eleman içeren verilerin saklanması (haftanın günleri, ay isimleri)
- Matematiksel işlemlerde sabit boyutlu veri kümeleri
- Algoritma alt yapılarında tampon (buffer) oluşturma
- Sıralama ve arama algoritmalarında geçici veri tutma

## Kullanım Türleri

```go
// 1. Eleman okuma ve yazma
gunler[0] = "Pazartesi"
ilkGun := gunler[0]

// 2. Dizi uzunluğunu öğrenme
uzunluk := len(gunler)

// 3. Döngü ile elemanlara erişme
for i := 0; i < len(gunler); i++ {
    fmt.Println(gunler[i])
}

// 4. range ile döngü (indeks ve değer)
for indeks, deger := range gunler {
    fmt.Println(indeks, deger)
}

// 5. Çok boyutlu diziler (matris)
var matris [2][3]int
matris[0][1] = 5
```

## Örnek Proje: Günlük Satış Toplamı Hesaplama

Bir kafenin haftalık satışlarını dizi ile saklayan ve toplam ciroyu hesaplayan örnek:

```go
package main

import "fmt" // Formatlı çıktı kütüphanesi

func main() {
    // Haftanın 7 günü için satış tutarlarını saklayan dizi
    satislar := [7]float64{1250.50, 980.25, 1420.00, 1105.75, 1890.90, 2240.10, 2010.60}

    // Gün isimlerini saklayan dizi
    gunler := [7]string{"Pazartesi", "Salı", "Çarşamba", "Perşembe", "Cuma", "Cumartesi", "Pazar"}

    // Toplam ciro için değişken tanımlama
    var toplam float64

    // Dizinin tüm elemanlarını döngü ile gezme
    for i := 0; i < len(satislar); i++ {
        // Her günün satışını ekrana yazdırma
        fmt.Printf("%s: %.2f TL\n", gunler[i], satislar[i])
        // Satış tutarını toplam değişkenine ekleme
        toplam += satislar[i]
    }

    // Haftanın en yüksek satışını bulma
    enYuksek := satislar[0] // İlk elemanı başlangıç değeri olarak alma
    for _, deger := range satislar {
        // Mevcut değer en yüksekten büyükse güncelleme
        if deger > enYuksek {
            enYuksek = deger
        }
    }

    // Haftalık toplam ciroyu yazdırma
    fmt.Printf("Haftalık Toplam Ciro: %.2f TL\n", toplam)
    // En yüksek günlük satışı yazdırma
    fmt.Printf("En Yüksek Günlük Satış: %.2f TL\n", enYuksek)
}
```

### Kod Çıktısı

```
Pazartesi: 1250.50 TL
Salı: 980.25 TL
Çarşamba: 1420.00 TL
Perşembe: 1105.75 TL
Cuma: 1890.90 TL
Cumartesi: 2240.10 TL
Pazar: 2010.60 TL
Haftalık Toplam Ciro: 10898.10 TL
En Yüksek Günlük Satış: 2240.10 TL
```

## Kütüphane Referansları

Kodda kullanılan `fmt` paketi Go standart kütüphanesine aittir. Detaylar: [fmt paketi](https://pkg.go.dev/fmt)

## Diziler ve Dilimler (Slices) Farkı

Dizilerin boyutu sabittir. Boyutu değişebilen yapılar için dilim (slice) kullanılır. Pratikte çoğu durumda dilim tercih edilir; boyutu bilinen sabit veriler için diziler daha verimlidir.

Kaynak: [Go Blog - Arrays, slices and strings](https://go.dev/blog/slices-intro), [Go Slices Dokümantasyonu](https://go.dev/tour/moretypes/7)

## Sonuç

Diziler, Go dilinde aynı türden sabit sayıda veriyi saklamanın temel yoludur. Boyutları tanımlandığı anda sabitlenir ve bellekte ardışık yer kaplar. Sabit boyutlu veriler için idealdir; dinamik boyut gerektiren durumlarda ise dilimler (slices) kullanılmalıdır. Dizileri iyi anlamak, dilimleri ve diğer koleksiyon yapılarını öğrenmek için önemli bir temeldir.
