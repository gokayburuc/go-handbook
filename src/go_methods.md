# GO - Methods

## Methods Nedir?

Go dilinde metot, belirli bir türe bağlanmış özel bir fonksiyondur. Normal fonksiyonlardan tek farkı, fonksiyon adından önce **alıcı (receiver)** adı verilen ek bir parametre almasıdır . Bu alıcı sayesinde fonksiyonlar özel bir türle gruplanır ve o türün davranışını tanımlar. Go'da sınıf (class) kavramı olmadığı için metotlar, struct'lara ve diğer kullanıcı tanımlı türlere davranış kazandırmanın temel yoludur .

Metotlar, Go'nun nesne yönelimli programlama (OOP) modeline yaklaşımının merkezinde yer alır. Kalıtım yerine kompozisyon ve gömme (embedding) kullanılır; metotlar bu yapılar üzerinden türlere yetenek kazandırır .

## Sözdizimi (Syntax)

Bir metot tanımlamak için `func` anahtar kelimesinden sonra parantez içinde alıcı belirtilir:

```go
func (aliciTipi aliciAdi) MetotAdi(parametreler) donusTipi {
    // metot gövdesi
}
```

Örnek:

```go
type Ucgen struct {
    Kenar int
}

// Ucgen türüne bagli cevre metodu
func (u Ucgen) Cevre() int {
    return u.Kenar * 3
}
```

Metot çağrısı, türün bir değeri üzerinden nokta operatörü ile yapılır: `u.Cevre()` .

## Kullanım Alanları

- **Türlere davranış kazandırma**: Struct veya kullanıcı tanımlı türlere hesaplama, doğrulama, dönüştürme gibi işlemler eklemek .
- **Arayüz (interface) uyumluluğu**: Bir türün arayüzü karşılaması için gereken metotları tanımlamak.
- **Kapsülleme**: Türün iç durumunu yalnızca metotlar aracılığıyla yönetmek.
- **Aynı isimli metotların farklı türlerde kullanımı**: `Ucgen.Cevre()` ve `Kare.Cevre()` gibi aynı isimli metotlar, farklı alıcı türlerinde bağımsız olarak tanımlanabilir .

## Kullanım Türleri

**1. Değer Alıcılı Metot (Value Receiver)**

Alıcı, türün bir kopyası olarak gelir. Metot içinde yapılan değişiklikler orijinal değere yansımaz .

```go
func (u Ucgen) Alan() int {
    return u.Kenar * u.Kenar / 2
}
```

**2. Pointer Alıcılı Metot (Pointer Receiver)**

Alıcı, türün bellek adresidir. Metot orijinal değeri değiştirebilir. Büyük struct'larda kopyalama maliyetini de düşürür .

```go
func (u *Ucgen) KenarIkiKatla() {
    u.Kenar *= 2 // orijinal deger degisir
}
```

**3. Gömülü Türlerin Metotlarını Kullanma**

Bir struct, başka bir türü gömerse, gömülü türün metotları dıştaki struct üzerinden çağrılabilir .

```go
type Sekil struct {
    Ucgen // gomulu alan
    Ad string
}

// s.Cevre() dogrudan Ucgen.Cevre() metodunu cagirir
```

**4. Metot İfadeleri (Method Expressions)**

Bir metot, alıcısı ilk parametre olacak şekilde normal bir fonksiyon gibi de kullanılabilir.

```go
cevreHesapla := Ucgen.Cevre // func(Ucgen) int
sonuc := cevreHesapla(Ucgen{Kenar: 5})
```

## Örnek Kod

Aşağıdaki örnek, bir banka hesabı türü üzerinde değer ve pointer alıcılı metotları göstermektedir.

```go
package main

import "fmt"

// Hesap, bir banka hesabini temsil eder.
type Hesap struct {
    Sahip  string
    Bakiye float64
}

// BakiyeOku, deger alicili metottur. Hesabin kopyasi uzerinde calisir.
// Sadece okuma yapar; orijinali degistirmez.
func (h Hesap) BakiyeOku() float64 {
    return h.Bakiye
}

// ParaYatir, pointer alicili metottur. Orijinal hesabi degistirir.
// Degisiklik cagiran tarafindan gorulur.
func (h *Hesap) ParaYatir(miktar float64) {
    h.Bakiye += miktar
}

// Ozet, hesap bilgilerini formatlar. Deger alicisi kullanir.
func (h Hesap) Ozet() string {
    return fmt.Sprintf("%s: %.2f TL", h.Sahip, h.Bakiye)
}

func main() {
    hesap := Hesap{Sahip: "Ayse", Bakiye: 1000.0}

    // Deger alicili metot cagrisi
    fmt.Println("Baslangic bakiyesi:", hesap.BakiyeOku())

    // Pointer alicili metot cagrisi
    hesap.ParaYatir(500.0)
    fmt.Println("Para yatirma sonrasi:", hesap.BakiyeOku())

    // Ozet metodu
    fmt.Println(hesap.Ozet())

    // Pointer uzerinden metot cagrisi
    hesapPtr := &hesap
    hesapPtr.ParaYatir(250.0)
    fmt.Println("Pointer ile yatirma sonrasi:", hesap.Bakiye)
}
```

**Çıktı:**

```
Baslangic bakiyesi: 1000
Para yatirma sonrasi: 1500
Ayse: 1500.00 TL
Pointer ile yatirma sonrasi: 1750
```

## En İyi Uygulamalar

- **Değişiklik gerekiyorsa pointer alıcı kullanın.** Metot alıcıyı değiştirecekse, alıcı **mutlaka** pointer olmalıdır .
- **Büyük struct'larda pointer alıcı tercih edin.** Kopyalama maliyetini düşürür .
- **Tutarlılık sağlayın.** Bir türün bazı metotları pointer alıcı kullanıyorsa, diğerleri de kullanmalıdır; aksi halde metot kümesi (method set) tutarsız olur .
- **Küçük türlerde değer alıcı yeterlidir.** Temel türler, küçük struct'lar ve dilimler için değer alıcı hem verimli hem de açıktır .
- **Aynı isimli metotları farklı türlerde kullanmaktan çekinmeyin.** Go'da bu doğaldır ve paket adı öneki gerektirmez .

> **Sık Yapılan Hatalar ve Uyarılar**
>
> **Değer alıcılı metotta değişiklik yapmak** orijinal değeri etkilemez; metot yalnızca kopya üzerinde çalışır. Alıcıyı değiştirmek istiyorsanız pointer alıcı kullanmalısınız . **Pointer alıcılı bir metodu, adreslenemeyen bir değer üzerinden çağırmak** derleme hatasına yol açabilir; örneğin bir map elemanı veya fonksiyon dönüş değeri üzerinden pointer metodu çağrılamaz. **Gömülü türün pointer alıcılı metotları**, dıştaki struct'ın pointer'ı üzerinden erişilebilir; değer üzerinden erişim her zaman mümkün olmayabilir.

## Kaynakça

- Microsoft Learn – Go'da yöntemleri kullanma: https://learn.microsoft.com/tr-tr/training/modules/go-methods-interfaces/1-methods
- Go FAQ – Pointer vs Value Receivers: https://go.dev/doc/faq#pointer_vs_value_receivers

## Sonuç

Go'da metotlar, belirli bir türe bağlanmış fonksiyonlardır ve alıcı parametresi ile normal fonksiyonlardan ayrılır. Değer alıcılı metotlar kopya üzerinde çalışırken, pointer alıcılı metotlar orijinal değeri değiştirebilir. Metotlar, struct'lara davranış kazandırmanın, arayüz uyumluluğu sağlamanın ve kapsülleme yapmanın temel aracıdır. Alıcı türü seçiminde değişiklik ihtiyacı, verimlilik ve tutarlılık göz önünde bulundurulmalıdır. Doğru kullanıldığında metotlar, Go kodunu daha düzenli ve okunabilir hale getirir.
