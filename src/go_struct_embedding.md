# GO - Structs Embedding

## Go'da Struct Gömme (Struct Embedding) Nedir?

Go dilinde struct gömme, bir struct'ın başka bir struct'ı **alan adı vermeden** içermesine olanak tanıyan bir kompozisyon mekanizmasıdır . Gömülen türün alanları ve metotları, dıştaki struct'a **terfi eder** (promoted); yani dıştaki struct üzerinden doğrudan erişilebilir hale gelir .

Go'da kalıtım (inheritance) yoktur. Struct gömme, diğer dillerdeki kalıtıma benzer bir kod paylaşımı sağlasa da **temel fark**, dıştaki struct'ın içteki struct'ın bir alt türü olmamasıdır . İlişki "is-a" (bir türüdür) değil, "has-a" (bir tane içerir) ilişkisidir .

## Sözdizimi (Syntax)

Gömme, struct tanımında alan adı yazılmadan türün doğrudan belirtilmesiyle yapılır:

```go
type Base struct {
    ID int
}

// Base, Container içine gömülür (alan adı yok)
type Container struct {
    Base        // gömülü alan (anonymous field)
    Name string
}
```

Gömülü alana başlatma sırasında tür adı alan adı olarak kullanılır:

```go
c := Container{
    Base: Base{ID: 1}, // tür adı = alan adı
    Name: "test",
}
```

Erişim iki şekilde yapılabilir: terfi eden doğrudan erişim (`c.ID`) veya açık yol (`c.Base.ID`) .

## Kullanım Alanları

- **Kod paylaşımı (Mixin)**: Ortak alan ve metotları birden fazla struct arasında paylaşmak .
- **Arayüz uyumluluğu**: Gömülen tür bir arayüzü uyguluyorsa, dıştaki tür de otomatik olarak o arayüzü uygular .
- **Dekoratör kalıbı**: Bir türü sararak ek davranış eklemek .
- **Arayüz birleştirme**: Küçük arayüzleri birleştirerek daha büyük arayüzler oluşturmak .

## Kullanım Türleri

**1. Temel Gömme ve Terfi**

Gömülen türün alanları ve metotları dıştaki türde doğrudan kullanılabilir .

```go
type base struct {
    num int
}

func (b base) describe() string {
    return fmt.Sprintf("base with num=%v", b.num)
}

type container struct {
    base
    str string
}

co := container{base: base{num: 1}, str: "some name"}
fmt.Println(co.num)        // terfi eden alan
fmt.Println(co.describe()) // terfi eden metot
```

**2. Arayüz Uyumluluğu**

Gömülen tür bir arayüzü karşılıyorsa, dıştaki tür de o arayüzü karşılar .

```go
type describer interface {
    describe() string
}

// container, base'i gömduğu için describer'ı da uygular
var d describer = co
```

**3. Çoklu Gömme**

Bir struct birden fazla türü gömebilir; tüm alan ve metotlar terfi eder .

```go
type Service struct {
    logger    // log(msg string) metodu terfi eder
    validator // validate() bool metodu terfi eder
    name      string
}
```

**4. Arayüz Gömme**

Arayüzler de birbirine gömülebilir; metot kümeleri birleşir .

```go
type ReadWriter interface {
    Reader
    Writer
}
```

## Örnek Kod

Aşağıdaki örnek, bir banka hesap sisteminde temel hesap işlemlerinin premium hesaba gömülmesini göstermektedir.

```go
package main

import "fmt"

// Hesap, temel banka hesabi islemlerini tutar.
type Hesap struct {
    Bakiye float64
}

// ParaYatir, hesaba para ekler.
func (h *Hesap) ParaYatir(miktar float64) {
    h.Bakiye += miktar
}

// BakiyeOku, mevcut bakiyeyi dondurur.
func (h Hesap) BakiyeOku() float64 {
    return h.Bakiye
}

// PremiumHesap, Hesap'i gomer ve ek ozellikler ekler.
// Hesap alanlari ve metotlari otomatik olarak terfi eder.
type PremiumHesap struct {
    Hesap           // gomulu alan
    FaizOrani float64
}

// FaizEkle, bakiyeye faiz uygular.
func (p *PremiumHesap) FaizEkle() {
    // Gomulu Hesap'in ParaYatir metodu dogrudan cagrilabilir
    p.ParaYatir(p.Bakiye * p.FaizOrani)
}

func main() {
    // PremiumHesap olusturma: gomulu alan tur adiyla baslatilir
    ph := PremiumHesap{
        Hesap:     Hesap{Bakiye: 1000.0},
        FaizOrani: 0.10,
    }

    // Terfi eden metot dogrudan cagrilir
    ph.ParaYatir(500.0)
    fmt.Printf("Para yatirma sonrasi: %.2f TL\n", ph.BakiyeOku())

    // Faiz uygula (gomulu metodu kullanir)
    ph.FaizEkle()
    fmt.Printf("Faiz sonrasi: %.2f TL\n", ph.BakiyeOku())

    // Acik yol ile erisim de mumkundur
    fmt.Printf("Acik erisim: %.2f TL\n", ph.Hesap.BakiyeOku())
}
```

**Çıktı:**

```
Para yatirma sonrasi: 1500.00 TL
Faiz sonrasi: 1650.00 TL
Acik erisim: 1650.00 TL
```

## En İyi Uygulamalar

- **Yalnızca API'yi açmak istediğinizde gömün.** Gömülen türün tüm metotları dıştaki türün genel API'sinin bir parçası olur. İç durumu gizlemek istiyorsanız isimli (küçük harfli) alan kullanın .
- **Gömme derinliğini 1-2 seviyeyle sınırlayın.** Çok katmanlı gömme, alan ve metot çakışmalarını belirsizleştirir ve okunabilirliği azaltır .
- **`sync.Mutex` gibi eşzamanlılık türlerini gömmekten kaçının.** Bu, `Lock()` ve `Unlock()` metotlarını dışarıya açar ve kapsüllemeyi bozar; isimli alan kullanın .
- **"Kalıtım" simülasyonu için kullanmayın.** Go'da tür hiyerarşisi yoktur; dıştaki tür içteki türe dönüştürülemez. Düz kompozisyon tercih edin .

> **Sık Yapılan Hatalar ve Uyarılar**
>
> **`sync.Mutex` gömme tuzağı**: `type Cache struct { sync.Mutex; items map[string]string }` tanımı, `cache.Lock()` çağrısına izin verir; bu da dışarıdan kilitleme yapılarak deadlock'a yol açabilir. Çözüm: `mu sync.Mutex` şeklinde isimli alan kullanmak . **Metot çakışması**: İki gömülü tür aynı isimde metoda sahipse ve dıştaki tür bu metodu çağırırsa, derleyici belirsizlik hatası verir; açık yol (`d.Base1.Method()`) kullanılmalıdır. **Gömülen türün alıcısı**: Terfi eden metot çağrıldığında, alıcı gömülü alanın bir kopyasıdır; pointer receiver kullanılmıyorsa dıştaki struct üzerinden yapılan değişiklikler içteki alana yansımaz .

## Kaynakça

- Go by Example – Struct Embedding: https://gobyexample.com/struct-embedding
- Effective Go – Embedding: https://go.dev/doc/effective_go#embedding
- Go Dil Spesifikasyonu – Struct types: https://go.dev/ref/spec#Struct_types

## Sonuç

Go'da struct gömme, kod paylaşımı ve kompozisyon için güçlü bir mekanizmadır. Gömülen türün alanları ve metotları dıştaki türe terfi eder; bu sayede kod tekrarı azalır ve arayüz uyumluluğu otomatik olarak sağlanır. Ancak gömme, kalıtım değildir: dıştaki tür içteki türün alt türü değildir ve tür dönüşümü yapılamaz. `sync.Mutex` gibi türleri gömmek yerine isimli alan kullanmak, kapsüllemeyi korur. Doğru kullanıldığında struct gömme, Go'nun kompozisyon odaklı tasarım felsefesini yansıtan temiz ve etkili bir araçtır.
