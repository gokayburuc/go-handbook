# GO - Values

# Go'da Değerler (Values)

## Değer Nedir?

Go'da her değişken bir değer (value) taşır. Değerler temel türlerde (int, float64, string, bool) veya bileşik türlerde saklanır. Go statik tipli bir dildir; her değişkenin türü derleme aşamasında bellidir. Tanımlanmış ama değer atanmamış değişkenlere sıfır değer (zero value) verilir.

Kaynak: [Go Turu - Values](https://go.dev/tour/basics/9)

## Söz Dizimi (Syntax)

```go
// Değişken tanımlama biçimleri

var sayi int              // Sıfır değeri 0

var fiyat float64 = 19.99 // Tür ve başlangıç değeri ile

var isim = "Gokay"        // Tür otomatik çıkarılır (type inference)

sehir := "Ankara"         // Kısa tanımlama, sadece fonksiyon içinde

const Pi = 3.14159        // Sabit tanımlama, değiştirilemez

var (
    a int     = 10        // Gruplu tanımlama
    b string  = "merhaba"
)
```

Temel türler ve sıfır değerleri:

| Tür          | Sıfır Değeri |
| ------------ | ------------ |
| int, float64 | `0`, `0.0`   |
| bool         | `false`      |
| string       | `""` (boş)   |

## Kullanım Alanları

- Sayısal hesaplamalar, sayaçlar ve fiyat bilgileri
- Metin ve kullanıcı verisi saklama
- Koşul ve durum kontrolü (`bool`)
- Sabitler ile sabit config değerleri tanımlama (`const`)
- Fonksiyonlar arasında veri aktarımı

## Kullanım Türleri

```go
// 1. Temel türlerle çalışma
var yasin int = 25
var aktif bool = true
var bakiye float64 = 1500.75

// 2. Tür dönüştürme (conversion), Go'da otomatik dönüştürme yoktur
tamSayi := int(bakiye)        // float64 -> int, ondalık kısmı atılır

// 3. Sabit tanımlama
const indirimOrani = 0.15

// 4. Tip doğrulama
var x interface{} = 42
if deger, ok := x.(int); ok {
    fmt.Println("Bu bir int:", deger)
}
```

## Örnek Proje: Banka Hesabı Bakiye Hesaplama

Basit bir banka hesabının bakiye işlemlerini değerlerle yöneten örnek:

```go
package main

import "fmt" // Formatlı çıktı kütüphanesi

// Hesap bilgilerini yazdıran yardımcı fonksiyon
func hesapBilgisi(isim string, bakiye float64, aktif bool) {
    // Değerleri formatlı şekilde yazdırma
    fmt.Printf("Hesap: %s | Bakiye: %.2f TL | Aktif: %t\n", isim, bakiye, aktif)
}

func main() {
    // Hesap sahibinin adı, string değer
    var musteri string = "Gokay Buruc"

    // Başlangıç bakiyesi, float64 değer
    var bakiye float64 = 5000.00

    // Hesabın aktiflik durumu, bool değer
    var aktif bool = true

    // Hesap bilgilerini yazdırma
    hesapBilgisi(musteri, bakiye, aktif)

    // Para yatırma: bakiye değerini güncelleme
    yatirilan := 2500.50 // kısa tanımlama ile yeni değer
    bakiye += yatirilan
    hesapBilgisi(musteri, bakiye, aktif)

    // Faiz oranı sabiti, const ile tanımlama
    const faizOrani = 0.10
    faiz := bakiye * faizOrani // faiz hesaplama
    fmt.Printf("Kazanılan Faiz: %.2f TL\n", faiz)

    // Tür dönüştürme: bakiyeyi tam sayıya çevirme
    // Ondalık kısmı atılır, yuvarlama yapılmaz
    tamBakiye := int(bakiye)
    fmt.Println("Tam Sayı Bakiye:", tamBakiye)

    // Hesabı dondurma: bool değeri değiştirme
    aktif = false
    hesapBilgisi(musteri, bakiye, aktif)
}
```

### Kod Çıktısı

```
Hesap: Gokay Buruc | Bakiye: 5000.00 TL | Aktif: true
Hesap: Gokay Buruc | Bakiye: 7500.50 TL | Aktif: true
Kazanılan Faiz: 750.05 TL
Tam Sayı Bakiye: 7500
Hesap: Gokay Buruc | Bakiye: 7500.50 TL | Aktif: false
```

## En İyi Uygulamalar (Best Practices)

- Fonksiyon içinde `:=` kısa tanımlamasını, paket düzeyinde `var` kullanın.
- Değişmeyen değerler için `const` tercih edin; bu, yanlışlıkla değiştirmeyi önler.
- Değişkenleri dar kapsamda tanımlayın ve kullanılmayanları hemen kaldırın.
- Tamsayı değerlerde gereksiz `int64` veya `uint` kullanmayın; çoğu durumda `int` yeterlidir.

## Sık Yapılan Hatalar ve Uyarılar

> **Uyarı:** Go'da tür dönüştürme otomatik değildir. `int` bir değerle `float64` bir değeri doğrudan toplamak derleme hatası verir; açık dönüşüm (`float64(x)`) gereklidir.

> **Uyarı:** Kullanılmayan değişkenler ve import edilmiş paketler derleme hatasına yol açar. Go bunu bilinçli olarak yapar; kod temizliğini zorunlu kılar.

> **Uyarı:** `:=` yalnızca fonksiyon gövdesi içinde çalışır; paket düzeyinde kullanılamaz.

> **Uyarı:** `int(float64)` dönüşümü yuvarlama yapmaz, ondalık kısmı keser. Doğru yuvarlama için `math.Round` kullanın.

Kaynak: [Go Turu - Basic types](https://go.dev/tour/basics/11), [math paketi](https://pkg.go.dev/math)

## Kütüphane Referansları

Kodda kullanılan `fmt` paketi Go standart kütüphanesine aittir. Detaylar: [fmt paketi](https://pkg.go.dev/fmt)

## Sonuç

Değerler, Go programlamanın temel yapı taşıdır. Statik tip sistemi sayesinde tür hataları derleme aşamasında yakalanır. Sıfır değer kavramı, tanımsız durumları güvenli kılar. `var`, `:=` ve `const` tanımlamalarını doğru yerde kullanmak ve tür dönüşümlerini açıkça yapmak, hatasız Go kodu yazmanın anahtarıdır.
