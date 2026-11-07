# GO - Variables

# Go'da Değişkenler (Variables)

## Değişken Nedir?

Değişken (variable), bir değeri bellekte saklamak için kullanılan isimlendirilmiş bir alandır. Go statik tipli bir dildir; her değişkenin türü bellidir. Değişkenler `var` anahtar kelimesi veya `:=` kısa tanımlaması ile oluşturulur. Tanımlanan ama değer atanmayan değişkenler sıfır değer (zero value) alır.

Kaynak: [Go Turu - Variables](https://go.dev/tour/basics/8)

## Söz Dizimi (Syntax)

```go
// Değişken tanımlama biçimleri

var sayi int              // Sadece tür, sıfır değeri 0

var fiyat float64 = 19.99 // Tür ve değer ile

var isim = "Gokay"        // Değer ile, tür otomatik çıkarılır

sehir := "Ankara"         // Kısa tanımlama, sadece fonksiyon içinde

var (
    a int    = 10         // Gruplu tanımlama
    b string = "merhaba"
)
```

Temel türlerin sıfır değerleri: sayısal türlerde `0`, `bool` için `false`, `string` için `""`.

## Kullanım Alanları

- Program boyunca değişen verilerin saklanması (sayaç, bakiye, durum)
- Fonksiyon sonuçlarının geçici olarak tutulması
- Kullanıcı girdisi ve hesaplama sonuçlarının taşınması
- Paket düzeyinde paylaşılan yapılandırma değerleri

## Kullanım Türleri

```go
// 1. Kısa tanımlama (fonksiyon içinde en yaygın yöntem)
toplam := 100

// 2. Değer atama ve güncelleme
toplam = toplam + 50
toplam += 25 // kısaltılmış biçim

// 3. Çoklu atama
x, y := 1, 2

// 4. Değişkenleri takas etme
x, y = y, x

// 5. Boş tanımlayıcı ile kullanılmayan değerleri yok sayma
deger, _ := hesapla() // ikinci dönüş değeri yok sayılır

// 6. Paket düzeyinde değişken
var surum = "1.0.0"
```

## Örnek Proje: Öğrenci Not Takip Uygulaması

Bir öğrencinin sınav notlarını değişkenlerle yöneten ve ortalamayı hesaplayan örnek:

```go
package main

import "fmt" // Formatlı çıktı kütüphanesi

// Paket düzeyinde sabit geçme notu
var gecmeNotu float64 = 50.0

func main() {
    // Öğrenci bilgileri, kısa tanımlama ile
    ogrenci := "Ayse Yilmaz"
    sinif := 11

    // Sınav notları
    vize := 65.5
    final := 78.25

    // Vize %40, final %60 ağırlığıyla ortalamayı hesaplama
    ortalama := vize*0.4 + final*0.6

    // Not bilgisini yazdırma
    fmt.Printf("Ogrenci: %s | Sinif: %d\n", ogrenci, sinif)
    fmt.Printf("Ortalama: %.2f\n", ortalama)

    // Ortalamanın geçme notuna göre durumunu kontrol etme
    // bool türünde değişkene atama
    gecti := ortalama >= gecmeNotu
    fmt.Println("Gecme Durumu:", gecti)

    // Sayıyı yuvarlayarak harf notu belirleme
    // int dönüşümü ondalığı keser
    yuvarlanmis := int(ortalama)
    fmt.Println("Yuvarlanmis Not:", yuvarlanmis)

    // Çoklu atama ile ek sınav notları
    not1, not2 := 90, 85
    fmt.Println("Ek Sinavlar:", not1, not2)
}
```

### Kod Çıktısı

```
Ogrenci: Ayse Yilmaz | Sinif: 11
Ortalama: 73.15
Gecme Durumu: true
Yuvarlanmis Not: 73
Ek Sinavlar: 90 85
```

## En İyi Uygulamalar (Best Practices)

- Fonksiyon içinde `:=` kısa tanımlamasını, paket düzeyinde `var` kullanın.
- Değişken adlarını kısa ama anlamlı verin; `kullaniciAdi` gibi.
- Kısa ömürlü değişkenleri kullanıldıkları yere yakın tanımlayın.
- Değişmeyecek değerler için `const` tercih edin.
- Boş tanımlayıcı `_` ile gereksiz dönüş değerlerini açıkça yok sayın.

## Sık Yapılan Hatalar ve Uyarılar

> **Uyarı:** Go'da kullanılmayan değişken derleme hatasıdır. Tanımladığınız her değişkeni kullanın ya da kaldırın.

> **Uyarı:** `:=` yalnızca fonksiyon gövdesi içinde çalışır, paket düzeyinde kullanılamaz.

> **Uyarı:** Aynı kapsamda `:=` ile aynı değişkeni yeniden tanımlamak derleme hatasıdır. Ancak çoklu tanımlamada en az bir yeni değişken varsa mevcut değişkenler yeniden atanabilir; bu davranış kafa karıştırabilir.

> **Uyarı:** Sıfır değeri tehlikesi: tanımsız kalan değişkenler `0` veya `false` olur; bu durum mantık hatası gibi sessizce yanlış sonuçlar üretebilir.

Kaynak: [Go Turu - Variables](https://go.dev/tour/basics/8), [Go Blog](https://go.dev/blog/)

## Kütüphane Referansları

Kodda kullanılan `fmt` paketi Go standart kütüphanesine aittir. Detaylar: [fmt paketi](https://pkg.go.dev/fmt)

## Sonuç

Değişkenler, Go'da veri saklamanın temel yoludur. `var` ve `:=` tanımlamaları ile tür çıkarımı, temiz ve kısa kod yazmayı sağlar. Statik tip sistemi ve kullanılmayan değişken yasağı, hataları erken aşamada yakalar. Sıfır değer kavrumunu anlamak ve doğru tanımlama yöntemini seçmek, sağlam Go kodu yazmanın temelidir.
