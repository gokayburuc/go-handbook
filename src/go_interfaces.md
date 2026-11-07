# GO - Interfaces

## Interface Nedir?

Go dilinde arayüz (interface), bir türün sahip olması gereken metot imzalarını tanımlayan **soyut bir türdür** . Arayüzler, farklı somut türlerin aynı arayüzü uygulayarak aynı şekilde ele alınmasını sağlar; bu da Go'da polimorfizmin temel yoludur .

Go'nun arayüzleri diğer dillerden önemli bir farklılık taşır: **örtük uygulama (implicit implementation)**. Bir türün arayüzü uyguladığını belirtmek için `implements` gibi bir anahtar kelime yoktur; tür, arayüzde tanımlanan tüm metotları içeriyorsa o arayüzü otomatik olarak karşılar . Bu tasarım, türler ile arayüzler arasında gevşek bağlılık sağlar.

## Sözdizimi (Syntax)

Bir arayüz, `type` ve `interface` anahtar kelimeleriyle tanımlanır:

```go
type Shape interface {
    Perimeter() float64
    Area() float64
}
```

Bu arayüz, `Shape` olarak kabul edilmek isteyen her türün hem `Perimeter()` hem de `Area()` metotlarını içermesi gerektiğini belirtir . Arayüz yalnızca **sözleşmeyi** tanımlar; metotların nasıl uygulanacağına dair bilgi içermez.

## Kullanım Alanları

- **Polimorfizm**: Farklı türlerin aynı arayüz üzerinden aynı şekilde çağrılmasını sağlamak .
- **Gevşek bağlılık (decoupling)**: Kodun somut uygulamalara değil, davranış sözleşmelerine bağımlı olmasını sağlamak .
- **Test edilebilirlik**: Arayüzler sayesinde gerçek uygulamalar yerine sahte (mock) nesneler kullanılabilir.
- **Standart kütüphane entegrasyonu**: `io.Reader`, `io.Writer`, `fmt.Stringer` gibi yerleşik arayüzler, özel türlerin standart kütüphaneyle uyumlu çalışmasını sağlar .

## Kullanım Türleri

**1. Temel Arayüz ve Örtük Uygulama**

Bir struct, arayüzdeki metotları tanımladığında arayüzü otomatik olarak karşılar .

```go
type Square struct {
    size float64
}

func (s Square) Area() float64      { return s.size * s.size }
func (s Square) Perimeter() float64 { return s.size * 4 }

// Square artık Shape arayüzünü uygular
var s Shape = Square{3}
```

**2. Boş Arayüz (`interface{}` veya `any`)**

Hiçbir metot içermeyen arayüz, **her tür** tarafından karşılanır. Bilinmeyen türdeki değerleri tutmak için kullanılır . Go 1.18'den itibaren `any`, `interface{}` için takma ad olarak tercih edilir .

```go
func printVal(v any) {
    fmt.Println(v)
}
```

**3. Tür Belirleme (Type Assertion)**

Arayüz değerinin altındaki somut türe erişmek için kullanılır. İki değerli form, türün eşleşip eşleşmediğini bildirir .

```go
if p, ok := s.(Person); ok {
    fmt.Println("Kişi:", p.name)
}
```

**4. Tür Anahtarı (Type Switch)**

Arayüz değerinin dinamik türüne göre farklı işlemler yapmak için `switch` kullanılır .

```go
switch v := x.(type) {
case int:
    fmt.Println("Tam sayı:", v)
case string:
    fmt.Println("Metin:", v)
default:
    fmt.Printf("Bilinmeyen: %T\n", v)
}
```

**5. Arayüz Gömme**

Bir arayüz, başka arayüzleri gömerek metot kümelerini birleştirebilir .

```go
type Reader interface { Read(p []byte) (int, error) }
type Writer interface { Write(p []byte) (int, error) }

type ReadWriter interface {
    Reader
    Writer
}
```

**6. `fmt.Stringer` ile Özel Yazdırma**

`String() string` metodunu tanımlayan türler, `fmt` paketi tarafından özel olarak yazdırılır .

```go
func (p Person) String() string {
    return fmt.Sprintf("%v (%v)", p.Name, p.Country)
}
```

## Örnek Kod

Aşağıdaki örnek, bir ödeme sisteminde farklı ödeme yöntemlerini ortak bir arayüz üzerinden yönetmeyi göstermektedir.

```go
package main

import "fmt"

// OdemeYontemi, tüm ödeme türlerinin uygulaması gereken sözleşmedir.
type OdemeYontemi interface {
	OdemeYap(tutar float64) string
	Ad() string
}

// KrediKarti, OdemeYontemi arayüzünü uygular.
type KrediKarti struct {
	KartNo string
}

func (k KrediKarti) OdemeYap(tutar float64) string {
	return fmt.Sprintf("%.2f TL kredi kartı (%s) ile ödendi", tutar, k.KartNo)
}

func (k KrediKarti) Ad() string { return "Kredi Kartı" }

// Havale, OdemeYontemi arayüzünü uygular.
type Havale struct {
	IBAN string
}

func (h Havale) OdemeYap(tutar float64) string {
	return fmt.Sprintf("%.2f TL havale (%s) ile ödendi", tutar, h.IBAN)
}

func (h Havale) Ad() string { return "Havale" }

// OdemeIsle, arayüz sayesinde hangi somut türün geldiğini bilmez.
func OdemeIsle(y OdemeYontemi, tutar float64) {
	fmt.Printf("[%s] %s\n", y.Ad(), y.OdemeYap(tutar))
}

func main() {
	// Farklı türler aynı arayüz üzerinden kullanılıyor
	yontemler := []OdemeYontemi{
		KrediKarti{KartNo: "1234-5678"},
		Havale{IBAN: "TR00 0000 0000"},
	}

	for _, y := range yontemler {
		OdemeIsle(y, 250.0)
	}

	// Tür anahtarı: arayüz değerinin somut türünü belirleme
	var x any = KrediKarti{KartNo: "9999"}
	switch v := x.(type) {
	case KrediKarti:
		fmt.Println("Tür anahtarı: KrediKartı, no:", v.KartNo)
	default:
		fmt.Printf("Bilinmeyen tür: %T\n", v)
	}
}
```

**Çıktı:**

```
[Kredi Kartı] 250.00 TL kredi kartı (1234-5678) ile ödendi
[Havale] 250.00 TL havale (TR00 0000 0000) ile ödendi
Tür anahtarı: KrediKartı, no: 9999
```

## En İyi Uygulamalar

- **Arayüzleri tüketen tarafta tanımlayın.** Arayüz, uygulamanın bulunduğu pakette değil, onu kullanan pakette tanımlanmalıdır; bu, gereksiz bağımlılıkları önler .
- **Küçük arayüzler tercih edin.** 1-3 metot içeren arayüzler daha kolay uygulanır ve test edilir. `io.Reader` ve `io.Writer` bunun en iyi örnekleridir .
- **Tek metotlu arayüzleri `-er` ekiyle adlandırın.** Go geleneğinde `Reader`, `Writer`, `Stringer` gibi isimler kullanılır .
- **Arayüz kabul edin, somut tür döndürün.** Fonksiyon parametreleri arayüz alabilir; ancak yapıcı (constructor) fonksiyonlar somut tür döndürmelidir .
- **`any` yerine tip parametrelerini (generics) değerlendirin.** Tür güvenliği gerektiğinde `interface{}` yerine Go 1.18+ tip parametreleri kullanın .

> **Sık Yapılan Hatalar ve Uyarılar**
>
> **Tek uygulamalı arayüzler gereksiz soyutlama yaratır.** Yalnızca bir somut türün uyguladığı ve testlerde mock'lanmayan arayüzler, fayda sağlamadan karmaşıklık ekler; somut türü doğrudan kullanmak daha temizdir . **Şişman arayüzler (fat interfaces)** uygulanması ve mock'lanması zor sözleşmeler oluşturur; ihtiyaç duyulan metotları küçük parçalara ayırın . **Değer alıcılı (value receiver) metotlar ile pointer alıcılı metotların metot kümeleri farklıdır.** `T` türünün metot kümesi yalnızca değer alıcılı metotları içerirken, `*T` hem değer hem pointer alıcılı metotları içerir; bu nedenle bir arayüzü yalnızca `*T` karşılayabilirken `T` karşılamayabilir . **`any` kullanımı tür güvenliğini kaybettirir**; `any` ile alınan değer üzerinde işlem yapmak için tür belirleme veya tür anahtarı gerekir, bu da çalışma zamanı hatalarına yol açabilir .

## Kaynakça

- Go Dil Spesifikasyonu – Interface types: https://go.dev/ref/spec#Interface_types
- A Tour of Go – Interfaces: https://go.dev/tour/methods/9
- Go FAQ – Method sets (T vs *T): https://go.dev/doc/faq#method_sets
- Microsoft Learn – Go'da Arayüzleri Kullanma: https://learn.microsoft.com/tr-tr/training/modules/go-methods-interfaces/2-interfaces

## Sonuç

Go'da arayüzler, türlerin davranışlarını sözleşme olarak tanımlayan ve örtük uygulama sayesinde gevşek bağlılık sağlayan temel bir yapıdır. Boş arayüz `any` her türü kabul ederken, küçük ve odaklı arayüzler daha esnek ve test edilebilir kod üretir. Tür belirleme ve tür anahtarı, arayüz değerlerinin somut türlerine erişmeyi sağlar. Arayüzleri tüketen tarafta tanımlamak, küçük tutmak ve `-er` ekiyle adlandırmak, sürdürülebilir Go kodunun temelini oluşturur.
