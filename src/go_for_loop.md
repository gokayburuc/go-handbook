# GO - For Loop

# Go'da Döngüler (For Loop)

## For Döngüsü Nedir?

For döngüsü, belirli bir kod bloğunu tekrar tekrar çalıştırmak için kullanılan kontrol yapısıdır. Go'da tek döngü yapısı `for`'dur; `while` ve `do-while` bulunmaz. `for` üç biçimde kullanılabilir: klasik üç parçalı döngü, koşullu döngü ve sonsuz döngü. Ayrıca koleksiyonlar için `range` ile kısa kullanım vardır.

Kaynak: [Go Turu - For](https://go.dev/tour/flowcontrol/1)

## Söz Dizimi (Syntax)

```go
// 1. Klasik üç parçalı döngü: başlangıç; koşul; artış
for i := 0; i < 5; i++ {
    fmt.Println(i)
}

// 2. Koşullu döngü (while gibi davranır)
for sayi < 10 {
    sayo++
}

// 3. Sonsuz döngü
for {
    // break ile çıkılır
}

// 4. range ile koleksiyon gezme
for indeks, deger := range dilim {
    fmt.Println(indeks, deger)
}
```

## Kullanım Alanları

- Listeler, dilimler, diziler ve haritalar üzerinde gezinme
- Belirli sayıda tekrar gereken işlemler (tablo üretme, sayaç)
- Dosya satırlarını veya veri akışlarını okuma
- Kullanıcı girdisi bekleme ve servis döngüleri (sonsuz döngü)
- Toplama, arama ve filtreleme işlemleri

## Kullanım Türleri

```go
// 1. range ile dilim gezme, indeks gerekmiyorsa boş tanımlayıcı
for _, urun := range urunler {
    fmt.Println(urun)
}

// 2. Harita gezme
for anahtar, deger := range harita {
    fmt.Println(anahtar, deger)
}

// 3. String gezme, rune bazlı
for i, r := range "merhaba" {
    fmt.Println(i, string(r))
}

// 4. break ile döngüden çıkma
for i := 0; i < 10; i++ {
    if i == 5 {
        break // döngüyü tamamen sonlandırır
    }
}

// 5. continue ile o turu atlama
for i := 0; i < 10; i++ {
    if i%2 == 0 {
        continue // sonraki tura geçer
    }
    fmt.Println(i) // sadece tek sayılar
}

// 6. Etiketli break ile iç içe döngüden çıkma
dis:
for i := 0; i < 3; i++ {
    for j := 0; j < 3; j++ {
        if i*j > 3 {
            break dis // dış döngüden çıkar
        }
    }
}
```

## Örnek Proje: Stok Kontrol ve Fiyat Toplamı

Bir marketin stok listesinde döngülerle işlem yapan örnek:

```go
package main

import "fmt" // Formatlı çıktı kütüphanesi

func main() {
    // Ürün isimlerini ve fiyatlarını tutan dilimler
    urunler := []string{"Elma", "Ekmek", "Sut", "Peynir"}
    fiyatlar := []float64{25.50, 10.00, 32.75, 180.90}

    // 1. Klasik for döngüsü ile tüm ürünleri listeleme
    fmt.Println("Fiyat Listesi:")
    for i := 0; i < len(urunler); i++ {
        // Ürün adını ve fiyatını yazdırma
        fmt.Printf("- %s: %.2f TL\n", urunler[i], fiyatlar[i])
    }

    // 2. range ile toplam sepet tutarını hesaplama
    var toplam float64
    for _, fiyat := range fiyatlar {
        // Her fiyatı toplama ekleme
        toplam += fiyat
    }
    fmt.Printf("Sepet Toplami: %.2f TL\n", toplam)

    // 3. Koşullu döngü ile bütçe kontrolü
    butce := 100.00 // müşteri bütçesi
    harcanan := 0.0
    indeks := 0
    for indeks < len(fiyatlar) {
        // Bütçe aşılacaksa döngüden çıkma
        if harcanan+fiyatlar[indeks] > butce {
            break
        }
        // Fiyatı harcanan tutara ekleme
        harcanan += fiyatlar[indeks]
        indeks++
    }
    fmt.Printf("Butce Icinde Alinan Urun: %d, Harcanan: %.2f TL\n", indeks, harcanan)

    // 4. continue ile sadece pahalı ürünleri bulma
    fmt.Println("100 TL Uzeri Urunler:")
    for i, fiyat := range fiyatlar {
        // Fiyat 100 TL'nin altındaysa bu turu atlama
        if fiyat <= 100 {
            continue
        }
        fmt.Printf("- %s: %.2f TL\n", urunler[i], fiyat)
    }
}
```

### Kod Çıktısı

```
Fiyat Listesi:
- Elma: 25.50 TL
- Ekmek: 10.00 TL
- Sut: 32.75 TL
- Peynir: 180.90 TL
Sepet Toplami: 249.15 TL
Butce Icinde Alinan Urun: 3, Harcanan: 68.25 TL
100 TL Uzeri Urunler:
- Peynir: 180.90 TL
```

## En İyi Uygulamalar (Best Practices)

- Koleksiyon gezme sırasında indeks gerekmiyorsa `for _, v := range xs` kullanın.
- Klasik döngü yerine `range` kullanmak indeks hatası riskini ortadan kaldırır.
- Döngü içinde gereksiz hesaplama yapmayın; sabitleri döngü dışına alın.
- Sonsuz döngülerde mutlaka bir çıkış koşulu (`break`, `return`) tanımlayın.
- Döngü sayaçları için `i, j, k` gibi kısa isimler gelenekseldir.

## Sık Yapılan Hatalar ve Uyarılar

> **Uyarı:** Go'da `while` döngüsü yoktur. Koşullu döngü için `for kosul { }` biçimi kullanılır.

> **Uyarı:** `range` her turda döngü değişkeninin bir kopyasını verir. Döngü içinde değişkeni değiştirmek koleksiyonu etkilemez.

> **Uyarı:** Go 1.22 öncesi sürümlerde `for` bloğu içinde tanımlanan değişkenler turlar arasında paylaşılırdı; closure (anonim fonksiyon) kullanımında beklenmeyen sonuçlar doğurabilir. Go 1.22 ve sonrasında her tur için yeni değişken oluşur.

> **Uyarı:** Etiketli `break` olmadan iç içe döngülerde yalnızca en içteki döngüden çıkılır.

Kaynak: [Go Turu - For continued](https://go.dev/tour/flowcontrol/2), [Go Turu - Range](https://go.dev/tour/moretypes/12)

## Kütüphane Referansları

Kodda kullanılan `fmt` paketi Go standart kütüphanesine aittir. Detaylar: [fmt paketi](https://pkg.go.dev/fmt)

## Sonuç

For döngüsü, Go'da tekrarlayan işlemlerin tek yapısal çözümüdür. Üç parçalı, koşullu ve sonsuz biçimleri ile `while` ve `do-while` ihtiyaçlarını karşılar. `range` ile koleksiyonlarda güvenli ve okunabilir gezinme sağlanır. `break`, `continue` ve etiketli döngülerle akış kontrolü tam anlamıyla yapılabilir. Döngü biçimini doğru seçmek, kodun hem okunabilirliğini hem de güvenliğini artırır.
