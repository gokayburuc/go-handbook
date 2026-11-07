# GO - Conditionals

# Go'da Koşullar (if - else if - else)

## Koşul İfadeleri Nedir?

Koşul ifadeleri, programın belirli bir duruma göre farklı kod bloklarını çalıştırmasını sağlar. `if` bir koşul doğruysa bloğu çalıştırır; `else if` alternatif koşulları, `else` hiçbir koşul doğru değilse çalışacak bloğu tanımlar. Koşullar `bool` değer döndürmelidir.

Kaynak: [Go Turu - If](https://go.dev/tour/flowcontrol/5)

## Söz Dizimi (Syntax)

```go
// Temel if yapısı
if yas >= 18 {
    fmt.Println("Yetiskin")
}

// if - else yapısı
if bakiye >= tutar {
    fmt.Println("Odeme yapildi")
} else {
    fmt.Println("Yetersiz bakiye")
}

// if - else if - else zinciri
if puan >= 85 {
    fmt.Println("AA")
} else if puan >= 70 {
    fmt.Println("BB")
} else {
    fmt.Println("CC")
}

// Kısa ifade ile if: değişken koşul içinde tanımlanır
if hata := islemYap(); hata != nil {
    fmt.Println("Hata:", hata)
}
```

## Kullanım Alanları

- Kullanıcı girdisi ve yetki kontrolü
- Hata yönetimi (`if err != nil`, Go'nun en yaygın kalıbı)
- Fiyat aralığına göre kampanya ve indirim seçimi
- Form doğrulama ve veri filtreleme
- Durum bazlı iş kuralları (puan, seviye, kategori)

## Kullanım Türleri

```go
// 1. Tek koşul
if sicaklik > 30 {
    fmt.Println("Hava cok sicak")
}

// 2. Mantıksal operatörler ile birleşik koşul
if yas >= 18 && ehliyetVar {
    fmt.Println("Arac kullanabilir")
}

// 3. Kısa ifade ile if, kapsam yalnızca if bloğu içindedir
if deger, err := strconv.Atoi(girdi); err == nil {
    fmt.Println("Gecerli sayi:", deger)
}

// 4. Olumsuz koşul
if !aktif {
    fmt.Println("Hesap pasif")
}

// 5. switch yerine geçebilecek kısa else-if zinciri
if saat < 12 {
    fmt.Println("Gunaydin")
} else if saat < 18 {
    fmt.Println("Iyi gunler")
} else {
    fmt.Println("Iyi aksamlar")
}
```

## Örnek Proje: Kredi Başvuru Değerlendirme

Bir bankanın kredi başvurusunu koşullarla değerlendiren örnek:

```go
package main

import "fmt" // Formatlı çıktı kütüphanesi

// Başvuru sonucunu değerlendiren fonksiyon
func krediDegerlendir(gelir float64, krediNotu int, mevcutBorc float64) string {
    // Gelir ve kredi notu birlikte kontrol ediliyor
    if krediNotu < 500 {
        // Düşük kredi notu doğrudan ret sebebi
        return "Red: Kredi notu cok dusuk"
    }

    // Mevcut borcun gelire oranı kontrol ediliyor
    if mevcutBorc > gelir*0.5 {
        // Borç oranı yüzde 50'nin üzerinde
        return "Red: Borc/gelir orani yuksek"
    }

    // Kredi notuna göre limit belirleme
    if krediNotu >= 800 && gelir >= 50000 {
        // En yüksek limit grubu
        return "Onay: Yuksek limit (Platinum musteri)"
    } else if krediNotu >= 650 {
        // Orta seviye limit grubu
        return "Onay: Standart limit"
    } else {
        // Not 500-650 arası, temkinli onay
        return "Onay: Dusuk limit"
    }
}

func main() {
    // Üç farklı müşteri başvurusu
    fmt.Println(krediDegerlendir(45000.0, 850, 5000.0))
    fmt.Println(krediDegerlendir(30000.0, 700, 12000.0))
    fmt.Println(krediDegerlendir(25000.0, 480, 0.0))

    // Kısa ifade ile if: sayı dönüşümü ve hata kontrolü
    // strconv kullanmadan basit bir sınır kontrolü
    yas := 22
    if v := yas - 18; v >= 0 {
        // v yalnızca bu blok içinde geçerli
        fmt.Printf("Yetiskin olali %d yil olmus\n", v)
    }
}
```

### Kod Çıktısı

```
Onay: Yukuk limit (Platinum musteri)
Onay: Standart limit
Red: Kredi notu cok dusuk
Yetiskin olali 4 yil olmus
```

## En İyi Uygulamalar (Best Practices)

- Önce olası hata durumunu kontrol edin: `if err != nil { return err }`. Bu, ana mantığı en altta tutar.
- Kısa ifade biçimini (`if v, ok := ...; kosul`) dönüş ve hata kontrolünde kullanın; değişken kapsamı dar kalır.
- `else` bloklarını azaltmak için erken dönüş (early return) tercih edin.
- `x == true` yazmayın; `if x` yeterlidir.
- Çok dallı zincirler için `switch` daha okunabilirdir; `else if` sayısı artıyorsa `switch`'e geçin.

## Sık Yapılan Hatalar ve Uyarılar

> **Uyarı:** Go'da koşul parantez içine alınmaz ve koşuldan sonra mutlaka süslü parantez gerekir: `if x { }`. Tek satırlık bloklar için bile parantez zorunludur.

> **Uyarı:** Kısa ifadede tanımlanan değişken yalnızca `if` ve `else` blokları içinde geçerlidir; dışarıda kullanılamaz.

> **Uyarı:** Mantıksal operatörleri doğru kullanın: `&&` (ve) ve `||` (veya). `&` ve `|` bitwise operatörlerdir, koşullarda hata üretir.

> **Uyarı:** `else if` zincirleri yukarıdan aşağı doğru ilk doğru koşulda durur. Koşul sırası yanlışsa, doğru olan dal hiç çalışmayabilir.

Kaynak: [Go Turu - If and else](https://go.dev/tour/flowcontrol/6), [Effective Go - If](https://go.dev/doc/effective_go#if)

## Kütüphane Referansları

Kodda kullanılan `fmt` paketi Go standart kütüphanesine aittir. Detaylar: [fmt paketi](https://pkg.go.dev/fmt)

## Sonuç

Koşul ifadeleri, programların karar verme mekanizmasının temelidir. Go'nun kısa ifade biçimi (`if v := ...; kosul`), değişken kapsamını daraltarak temiz kod yazmayı sağlar. Erken dönüş ve hata öncelikli kontrol kalıbı, Go ekosisteminin standartıdır. Basit dallanmalarda `if-else`, çoklu durumda `switch` tercih edilmelidir.
