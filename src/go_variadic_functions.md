# GO - Variadic Functions

## Variadic Function Nedir?

Go dilinde değişken argümanlı fonksiyon (variadic function), **son parametresi** özel `...` sözdizimi ile tanımlanan ve çağrılırken bu parametre için **sıfır veya daha fazla** argüman alabilen fonksiyondur . Fonksiyon içinde bu parametre, belirtilen türde bir **slice** olarak davranır; yani `func sum(nums ...int)` tanımında `nums` değişkeni `[]int` türündedir . Standart kütüphanedeki `fmt.Println` ve `fmt.Printf` en bilinen değişken argümanlı fonksiyon örnekleridir .

## Sözdizimi (Syntax)

Değişken argümanlı bir fonksiyon şu şekilde tanımlanır:

```go
func fonksiyonAdi(sabitParametreler, degiskenParametre ...T) dönüsTipi
```

`...T` ifadesi, "bu türden istediğiniz kadar değer gelebilir" anlamına gelir. Kritik kural: **`...T` parametresi her zaman parametre listesinin en sonunda yer almalıdır** .

```go
// Doğru: değişken parametre sonda
func logla(prefix string, degerler ...int)

// Yanlış: derleme hatası verir
func hatali(degerler ...int, prefix string)
```

## Kullanım Alanları

- **Toplu işlemler**: Birden fazla değeri tek çağrıda işlemek (toplama, birleştirme, filtreleme).
- **Loglama ve formatlama**: `fmt.Printf` gibi esnek argüman sayısı gerektiren çıktı fonksiyonları.
- **Yapılandırıcı fonksiyonlar (functional options)**: İsteğe bağlı ayarları değişken argümanlarla almak.
- **Sarıcı (wrapper) fonksiyonlar**: Alt fonksiyona değişken sayıda argümanı olduğu gibi iletmek.

## Kullanım Türleri

**1. Doğrudan Çağrı**: Argümanlar tek tek yazılır.

```go
topla(1, 2, 3)
```

**2. Spread Operatörü (`...`)**: Elde bir slice varsa, elemanları ayrı argümanlarmış gibi iletmek için çağrı sırasında `slice...` yazılır .

```go
sayilar := []int{1, 2, 3}
topla(sayilar...)
```

**3. Sıfır Argüman**: Değişken parametreye hiç değer verilmezse, fonksiyon içinde bu parametre **boş bir slice** (`[]T{}`) olur; `nil` değildir ve `range` ile güvenle gezilebilir .

**4. Alt Fonksiyona Aktarma**: Bir fonksiyonun değişken parametresini başka bir fonksiyona aynen iletmek için `...` kullanılır.

```go
func ustTopla(base int, ekler ...int) int {
    return base + topla(ekler...) // slice'ı spread ile ilet
}
```

## Örnek Kod

Aşağıdaki örnek, bir e-ticaret sepetindeki ürün fiyatlarını toplayan ve isteğe bağlı indirimleri uygulayan gerçekçi bir senaryoyu göstermektedir.

```go
package main

import "fmt"

// SepetToplami, temel ürün fiyatlarını ve isteğe bağlı indirimleri toplar.
// degiskenIndirimler ...float64: sıfır veya daha fazla indirim tutarı alabilir.
func SepetToplami(urunFiyatlari ...float64) float64 {
	toplam := 0.0
	// Fonksiyon içinde urunFiyatlari bir []float64 slice'ıdır.
	for _, fiyat := range urunFiyatlari {
		toplam += fiyat
	}
	return toplam
}

// IndirimUygula, temel toplamdan verilen indirimleri düşer.
// İndirim toplamı, temel toplamı aşarsa sonuç 0 olur.
func IndirimUygula(temelToplam float64, indirimler ...float64) float64 {
	toplamIndirim := 0.0
	for _, indirim := range indirimler {
		toplamIndirim += indirim
	}
	sonuc := temelToplam - toplamIndirim
	if sonuc < 0 {
		return 0
	}
	return sonuc
}

func main() {
	// Doğrudan çağrı: üç ürün fiyatı
	sepet1 := SepetToplami(100.0, 250.0, 75.0)
	fmt.Printf("Sepet 1 toplamı: %.2f TL\n", sepet1)

	// Sıfır argüman: boş sepet
	sepet2 := SepetToplami()
	fmt.Printf("Sepet 2 toplamı: %.2f TL\n", sepet2)

	// Slice'ı spread operatörü ile iletme
	fiyatlar := []float64{300.0, 150.0, 50.0}
	sepet3 := SepetToplami(fiyatlar...)
	fmt.Printf("Sepet 3 toplamı: %.2f TL\n", sepet3)

	// Değişken argümanları alt fonksiyona aktarma
	indirimler := []float64{20.0, 30.0}
	odenecek := IndirimUygula(sepet3, indirimler...)
	fmt.Printf("İndirimli ödenecek tutar: %.2f TL\n", odenecek)

	// Tek argümanlı indirim
	fmt.Printf("Tek indirimli tutar: %.2f TL\n", IndirimUygula(sepet1, 50.0))
}
```

**Çıktı:**

```
Sepet 1 toplamı: 425.00 TL
Sepet 2 toplamı: 0.00 TL
Sepet 3 toplamı: 500.00 TL
İndirimli ödenecek tutar: 450.00 TL
Tek indirimli tutar: 375.00 TL
```

## En İyi Uygulamalar

- **Değişken parametreyi her zaman en sona koyun.** Bu, dilin zorunlu bir kuralıdır; aksi halde derleme hatası alınır .
- **Fonksiyon içinde slice gibi davranın.** `len`, `range` ve indeksleme doğal olarak çalışır; ayrıca bir slice'a ihtiyaç duyduğunuzda doğrudan değişken parametreyi kullanabilirsiniz .
- **Slice'ı iletirken `...` eklemeyi unutmayın.** `topla(sayilar)` yazmak derleme hatası verir; doğrusu `topla(sayilar...)` şeklindedir .
- **Boş çağrıyı güvenli varsayın.** Değişken parametre sıfır argümanla çağrıldığında boş slice olur; `range` güvenlidir ve panik oluşmaz .

> **Sık Yapılan Hatalar ve Uyarılar**
>
> Değişken argümanlı bir fonksiyona **slice'ı spread operatörü olmadan** iletmek derleme hatasına yol açar; `topla(sayilar)` değil, `topla(sayilar...)` yazılmalıdır . Değişken parametre, fonksiyon içinde `nil` değil **boş slice** olarak gelir; yine de `len` ve `range` ile kontrol edilmesi güvenlidir. Bazı kütüphaneler değişken argümanı "isteğe bağlı" gibi sunsa da, içinde argüman sayısını kontrol edip **panic** fırlatabilir; bu tür fonksiyonları çağırırken belgelenen argüman sayısına uyun .

## Kaynakça

- Go Dil Spesifikasyonu – Function types (Variadic): https://go.dev/ref/spec#Function_types
- Go by Example – Variadic Functions: https://gobyexample.com/variadic-functions
- Effective Go – Variadic functions: https://go.dev/doc/effective_go#variadic

## Sonuç

Go'da değişken argümanlı fonksiyonlar, `...T` sözdizimi ile son parametrenin sıfır veya daha fazla değer almasını sağlar. Fonksiyon içinde bu parametre `[]T` türünde bir slice'tır. Spread operatörü (`slice...`) ile mevcut bir slice'ı ayrı argümanlarmış gibi iletebilirsiniz. Değişken parametre her zaman en sonda olmalıdır. Doğru kullanıldığında, esnek argüman sayısı gerektiren API'ler ve toplu işlemler için kod tekrarını azaltan temiz bir çözüm sunar.
