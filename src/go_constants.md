# GO - Constants

# Go'da Sabitler (Constants)

## Sabit Nedir?

Sabit (constant), programın çalışması boyunca değeri hiç değişmeyen, derleme zamanında belirlenen isimlendirilmiş değerdir. `const` anahtar kelimesi ile tanımlanır. Sabitlere atanan değer derleme aşamasında hesaplanabilir olmalıdır. Sabitler değiştirilemez; bu nedenle yanlışlıkla değiştirilmesi istenmeyen değerler için güvenlidir.

Kaynak: [Go Turu - Constants](https://go.dev/tour/basics/15)

## Söz Dizimi (Syntax)

```go
// Sabit tanımlama biçimleri

const Pi = 3.14159          // Tür otomatik çıkarılır

const Selamlama string = "Merhaba" // Tür açıkça belirtilmiş

const (
    // Gruplu sabit tanımlama
    DurumAktif  = 1
    DurumPasif  = 0
)

const (
    // iota ile otomatik artan değerler
    Pzt = iota // 0
    Sal        // 1
    Car        // 2
)
```

## Kullanım Alanları

- Matematiksel sabitler (Pi, Euler sayısı)
- Yapılandırma değerleri (dosya yolları, limitler, portlar)
- Durum kodları ve enum benzeri tanımlamalar (`iota` ile)
- Hata mesajları ve sabit metinler
- Birim çarpanları (KB, MB, GB)

## Kullanım Türleri

```go
// 1. Tip güvenli sabitler
const maxKullanici int = 100

// 2. iota ile otomatik numaralandırma
const (
    Ocak = iota + 1 // 1'den başlar
    Subat           // 2
    Mart            // 3
)

// 3. Bit kaydırma ile bayrak tanımlama
const (
    OkumaYetkisi  = 1 << iota // 1
    YazmaYetkisi               // 2
    SilmeYetkisi               // 4
)

// 4. Türsüz sabitler, bağlama göre farklı türler gibi davranır
const Buyuk = 1 << 40

// 5. Sabit ifadeler derleme zamanında hesaplanır
const toplam = 3 * 60 * 60 // saniye cinsinden 3 saat
```

## Örnek Proje: Kargo Ücret Hesaplama Uygulaması

Bir kargo firmasının sabit tarife değerleriyle ücret hesaplayan örnek:

```go
package main

import "fmt" // Formatlı çıktı kütüphanesi

// Kargo tarifesindeki sabit değerler, gruplu tanımlama
const (
    TabanUcret    float64 = 50.0 // Her gönderinin taban ücreti
    KgBasiUcret   float64 = 12.5 // Kilogram başına ücret
    MinPaketAgirligi  = 1        // Minimum ağırlık, kg cinsinden
    MaxPaketAgirligi  = 30       // Maksimum ağırlık, kg cinsinden
)

// Gönderi durumları için iota ile otomatik numaralandırma
const (
    Hazirlaniyor = iota // 0
    Yolda               // 1
    TeslimEdildi        // 2
)

func main() {
    // Gönderi bilgileri
    agirlik := 8
    durum := Yolda

    // Ağırlık sınır kontrolü
    if agirlik < MinPaketAgirligi || agirlik > MaxPaketAgirligi {
        fmt.Println("Gecersiz agirlik araligi")
        return
    }

    // Ücret hesaplama: taban ücret + kg başına ücret
    ucret := TabanUcret + float64(agirlik)*KgBasiUcret
    fmt.Printf("Agirlik: %d kg | Ucret: %.2f TL\n", agirlik, ucret)

    // Gönderi durumunu yazdırma
    fmt.Println("Durum Kodu:", durum)

    // Duruma göre mesaj seçme
    if durum == TeslimEdildi {
        fmt.Println("Gonderi teslim edildi")
    } else {
        fmt.Println("Gonderi hala yolda")
    }

    // Sabit ifadeler derleme zamanında hesaplanır
    const gunlukKapasite = MaxPaketAgirligi * 10
    fmt.Println("Gunluk Kapasite:", gunlukKapasite)
}
```

### Kod Çıktısı

```
Agirlik: 8 kg | Ucret: 150.00 TL
Durum Kodu: 1
Gonderi hala yolda
Gunluk Kapasite: 300
```

## En İyi Uygulamalar (Best Practices)

- Sabit adlarını `PascalCase` veya `CamelCase` ile yazın; Go'da sabitler için `SCREAMING_SNAKE_CASE` zorunlu değildir.
- Sabitleri gruplamak için parantezli `const` blokları kullanın.
- Enum benzeri değerler için `iota` tercih edin; elle numara yazmaktan kaçının.
- Paket düzeyinde paylaşılan sabitleri ilgili türün yanında tanımlayın.
- Değişmeyecek her değer için `const` kullanın; bu, derleyicinin optimizasyon yapmasını da sağlar.

## Sık Yapılan Hatalar ve Uyarılar

> **Uyarı:** Sabitlere çalışma zamanında hesaplanan değer atanamaz. `const x = time.Now()` derleme hatasıdır; sabit ifadeler derleme zamanında hesaplanabilir olmalıdır. Bu durumda değişken kullanın.

> **Uyarı:** Sabitler değiştirilemez. `const` ile tanımlanan değere yeniden atama yapmak derleme hatasıdır.

> **Uyarı:** Sabit tanımı `:=` kısa tanımlaması ile yapılamaz; `const x = 5` biçimi kullanılmalıdır.

> **Uyarı:** `iota` yalnızca `const` blokları içinde çalışır ve her satırda bir artar. Ara değer atlanması isteniyorsa boş tanımlayıcı (`_`) kullanın.

Kaynak: [Go Turu - Constants](https://go.dev/tour/basics/15), [Go Turu - Numeric Constants](https://go.dev/tour/basics/16)

## Kütüphane Referansları

Kodda kullanılan `fmt` paketi Go standart kütüphanesine aittir. Detaylar: [fmt paketi](https://pkg.go.dev/fmt)

## Sonuç

Sabitler, Go'da değişmeyen değerleri güvenle saklamanın yoludur. Derleme zamanında belirlenmeleri sayesinde yanlış atamalar derlenirken yakalanır. `iota` ile otomatik numaralandırma, enum benzeri yapıları temiz biçimde kurmayı sağlar. Değişken ile sabiti doğru ayırmak, hem kod okunabilirliğini hem de güvenliği artırır. Kural basittir: değeri hiç değişmeyecekse `const` kullanın.
