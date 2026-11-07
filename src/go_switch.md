# GO - Switch

## Switch Nedir?

Go dilinde `switch` deyimi, bir ifadenin değerini birden fazla `case` bloğuyla karşılaştırarak dallanma sağlayan bir kontrol yapısıdır. Uzun `if-else` zincirlerini daha okunabilir ve düzenli hale getirmek için kullanılır. Go'nun `switch` yapısı C, C++ veya Java gibi dillerden önemli farklılıklar taşır: her `case` bloğunun sonunda `break` yazmaya gerek yoktur; eşleşme sağlandığında `switch` otomatik olarak sonlanır .

Go'da `switch`, yalnızca tam sayılar üzerinde değil, string, boolean ve hatta interface türleri üzerinde de çalışabilir .

## Sözdizimi (Syntax)

Temel `switch` yapısı şu şekildedir:

```go
switch ifade {
case deger1:
    // ifade == deger1 ise çalışır
case deger2:
    // ifade == deger2 ise çalışır
default:
    // hiçbir case eşleşmezse çalışır (isteğe bağlı)
}
```

Go'da `switch` ifadesi parantez gerektirmez. Ayrıca `case` içinde virgülle ayrılmış birden fazla değer belirtilebilir: `case "A", "B":` .

## Kullanım Alanları

`switch` deyimi özellikle şu durumlarda tercih edilir:

- **Çoklu değer eşleştirme**: Bir değişkenin belirli sabit değerlerden hangisine eşit olduğunu kontrol etmek.
- **Durum makineleri**: Uygulama durumlarını (örneğin sipariş durumu, bağlantı durumu) yönetmek.
- **Tür ayrımı (Type Switch)**: `interface{}` türündeki bir değerin gerçek türünü belirlemek.
- **Koşul zincirlerini sadeleştirme**: Uzun `if-else if-else` zincirlerini daha okunabilir hale getirmek.

## Kullanım Türleri

Go'da `switch` deyiminin dört temel kullanım biçimi vardır.

**1. Değer Eşleştirmeli Switch**: Bir ifadenin değerini `case` değerleriyle karşılaştırır.

```go
switch gun {
case "Cumartesi", "Pazar":
    fmt.Println("Hafta sonu")
default:
    fmt.Println("Hafta içi")
}
```

**2. Koşullu Switch (İfadesiz)**: `switch` ifadesi boş bırakılırsa, `switch true` anlamına gelir. Her `case` bir boolean ifade içerir. Bu, `if-else` zincirinin temiz bir alternatifidir .

```go
switch {
case puan >= 90:
    fmt.Println("A")
case puan >= 80:
    fmt.Println("B")
default:
    fmt.Println("C")
}
```

**3. Tür Switch**: Bir interface değerinin dinamik türünü belirlemek için kullanılır. Yalnızca interface türündeki değerler üzerinde çalışır .

```go
switch v := x.(type) {
case int:
    fmt.Println("tam sayı", v)
case string:
    fmt.Println("metin", v)
default:
    fmt.Printf("bilinmeyen tür: %T\n", v)
}
```

**4. Fallthrough Kullanımı**: Go'da `case` blokları varsayılan olarak bir sonrakine geçmez. Bir sonraki `case` bloğunun da çalışmasını istiyorsanız `fallthrough` anahtar kelimesini kullanmalısınız .

## Örnek Kod

Aşağıdaki örnek, bir e-ticaret sipariş durumunu yöneten gerçekçi bir senaryoyu göstermektedir.

```go
package main

import (
	"fmt"
	"time"
)

// SiparisDurumu, siparişin mevcut durumunu temsil eder.
type SiparisDurumu int

const (
	Beklemede SiparisDurumu = iota
	Hazirlaniyor
	Kargoda
	TeslimEdildi
	IptalEdildi
)

// DurumMesaji, sipariş durumuna göre kullanıcıya gösterilecek mesajı döner.
func DurumMesaji(durum SiparisDurumu) string {
	// Değer eşleştirmeli switch kullanımı
	switch durum {
	case Beklemede:
		return "Siparişiniz alındı, onay bekleniyor."
	case Hazirlaniyor:
		return "Siparişiniz hazırlanıyor."
	case Kargoda:
		return "Siparişiniz kargoya verildi."
	case TeslimEdildi:
		return "Siparişiniz teslim edildi."
	case IptalEdildi:
		return "Siparişiniz iptal edildi."
	default:
		return "Bilinmeyen durum."
	}
}

// KargoSuresi, sipariş durumuna göre tahmini kargo süresini hesaplar.
// Koşullu switch (ifadesiz switch) kullanımını gösterir.
func KargoSuresi(durum SiparisDurumu) time.Duration {
	switch {
	case durum == Kargoda:
		return 2 * 24 * time.Hour // 2 gün
	case durum == Hazirlaniyor:
		return 1 * 24 * time.Hour // 1 gün
	case durum == TeslimEdildi || durum == IptalEdildi:
		return 0 // Süre yok
	default:
		return 3 * 24 * time.Hour // 3 gün
	}
}

// DegerTipi, interface değerinin türünü belirlemek için tür switch kullanır.
func DegerTipi(x interface{}) {
	switch v := x.(type) {
	case int:
		fmt.Printf("Tam sayı: %d\n", v)
	case string:
		fmt.Printf("Metin: %s\n", v)
	case bool:
		fmt.Printf("Boolean: %t\n", v)
	default:
		fmt.Printf("Bilinmeyen tür: %T\n", v)
	}
}

func main() {
	siparis := Kargoda

	// Değer eşleştirmeli switch örneği
	fmt.Println("Durum:", DurumMesaji(siparis))

	// Koşullu switch örneği
	sure := KargoSuresi(siparis)
	fmt.Printf("Tahmini kargo süresi: %.0f gün\n", sure.Hours()/24)

	// Tür switch örneği
	DegerTipi(42)
	DegerTipi("merhaba")
	DegerTipi(true)

	// Fallthrough örneği: Durum kodları sırasıyla birikimli işlenir.
	seviye := 2
	fmt.Print("Seviye açıklaması: ")
	switch seviye {
	case 1:
		fmt.Print("Başlangıç ")
		fallthrough
	case 2:
		fmt.Print("Orta ")
		fallthrough
	case 3:
		fmt.Print("İleri ")
	default:
		fmt.Println("seviye")
	}
	// Çıktı: Seviye açıklaması: Orta İleri seviye
}
```

**Çıktı:**

```
Durum: Siparişiniz kargoya verildi.
Tahmini kargo süresi: 2 gün
Tam sayı: 42
Metin: merhaba
Boolean: true
Seviye açıklaması: Orta İleri seviye
```

## En İyi Uygulamalar

- **Çoklu değerleri virgülle birleştirin.** Aynı işlemi yapan birden fazla `case` için `case 1, 2, 3:` yazın; `fallthrough` kullanmaktan kaçının .
- **İfadesiz `switch`'i `if-else` zincirleri için tercih edin.** Koşullu mantık içeren uzun `if-else` yapıları yerine ifadesiz `switch` daha okunabilirdir .
- **Tür switch'te özel türleri genel türlerden önce yazın.** Bir somut tür, bir interface'i implemente ediyorsa, interface `case`'i daha genel olduğu için önce yazılırsa somut tür `case`'ine asla ulaşılamaz .
- **`default` bloğunu bilinçli kullanın.** `default` zorunlu değildir; ancak beklenmeyen değerleri yakalamak veya hata durumunu ele almak için faydalıdır.

> **Sık Yapılan Hatalar ve Uyarılar**
>
> Go'da her `case` sonunda `break` yazmanıza gerek yoktur; `break` yazmak gereksizdir ve statik analiz araçları tarafından uyarı olarak işaretlenebilir . `fallthrough` anahtar kelimesi **koşulsuz** olarak bir sonraki `case` bloğuna geçer; bir sonraki `case`'in koşulunu kontrol etmez. Ayrıca `fallthrough`, `case` bloğunun son ifadesi olmalıdır; bir `if` bloğunun içinde kullanılamaz . Tür switch'te `fallthrough` **kullanılamaz** . Bir döngü içindeki `switch` bloğunda `break` kullanmak yalnızca `switch`'ten çıkar, döngüden çıkmaz; döngüden çıkmak için etiketli `break` kullanmalısınız.

## Kaynakça

- Go Resmi Dokümantasyonu – Switch Statements: https://go.dev/ref/spec#Switch_statements
- Go Wiki – Switch: https://go.dev/wiki/Switch
- Effective Go – Switch: https://go.dev/doc/effective_go#switch

## Sonuç

Go'da `switch` deyimi, çoklu koşul dallanmasını temiz ve öz bir şekilde ifade etmenin güçlü bir yoludur. Otomatik `break` davranışı sayesinde C benzeri dillerdeki yaygın hatalar önlenir. Değer eşleştirme, koşullu dallanma ve tür ayrımı gibi farklı kullanım biçimleriyle geniş bir yelpazeye hitap eder. `fallthrough` dikkatli kullanılmalı, tür switch'te ise `case` sıralaması doğru yapılmalıdır. Doğru kullanıldığında `switch`, kodun okunabilirliğini ve bakım kolaylığını önemli ölçüde artırır. Go'da Switch Deyimi

## Nedir?

Go dilinde `switch` deyimi, bir ifadenin değerini birden fazla `case` bloğuyla karşılaştırarak dallanma sağlayan bir kontrol yapısıdır. Uzun `if-else` zincirlerini daha okunabilir ve düzenli hale getirmek için kullanılır. Go'nun `switch` yapısı C, C++ veya Java gibi dillerden önemli farklılıklar taşır: her `case` bloğunun sonunda `break` yazmaya gerek yoktur; eşleşme sağlandığında `switch` otomatik olarak sonlanır .

Go'da `switch`, yalnızca tam sayılar üzerinde değil, string, boolean ve hatta interface türleri üzerinde de çalışabilir .

## Sözdizimi (Syntax)

Temel `switch` yapısı şu şekildedir:

```go
switch ifade {
case deger1:
    // ifade == deger1 ise çalışır
case deger2:
    // ifade == deger2 ise çalışır
default:
    // hiçbir case eşleşmezse çalışır (isteğe bağlı)
}
```

Go'da `switch` ifadesi parantez gerektirmez. Ayrıca `case` içinde virgülle ayrılmış birden fazla değer belirtilebilir: `case "A", "B":` .

## Kullanım Alanları

`switch` deyimi özellikle şu durumlarda tercih edilir:

- **Çoklu değer eşleştirme**: Bir değişkenin belirli sabit değerlerden hangisine eşit olduğunu kontrol etmek.
- **Durum makineleri**: Uygulama durumlarını (örneğin sipariş durumu, bağlantı durumu) yönetmek.
- **Tür ayrımı (Type Switch)**: `interface{}` türündeki bir değerin gerçek türünü belirlemek.
- **Koşul zincirlerini sadeleştirme**: Uzun `if-else if-else` zincirlerini daha okunabilir hale getirmek.

## Kullanım Türleri

Go'da `switch` deyiminin dört temel kullanım biçimi vardır.

**1. Değer Eşleştirmeli Switch**: Bir ifadenin değerini `case` değerleriyle karşılaştırır.

```go
switch gun {
case "Cumartesi", "Pazar":
    fmt.Println("Hafta sonu")
default:
    fmt.Println("Hafta içi")
}
```

**2. Koşullu Switch (İfadesiz)**: `switch` ifadesi boş bırakılırsa, `switch true` anlamına gelir. Her `case` bir boolean ifade içerir. Bu, `if-else` zincirinin temiz bir alternatifidir .

```go
switch {
case puan >= 90:
    fmt.Println("A")
case puan >= 80:
    fmt.Println("B")
default:
    fmt.Println("C")
}
```

**3. Tür Switch**: Bir interface değerinin dinamik türünü belirlemek için kullanılır. Yalnızca interface türündeki değerler üzerinde çalışır .

```go
switch v := x.(type) {
case int:
    fmt.Println("tam sayı", v)
case string:
    fmt.Println("metin", v)
default:
    fmt.Printf("bilinmeyen tür: %T\n", v)
}
```

**4. Fallthrough Kullanımı**: Go'da `case` blokları varsayılan olarak bir sonrakine geçmez. Bir sonraki `case` bloğunun da çalışmasını istiyorsanız `fallthrough` anahtar kelimesini kullanmalısınız .

## Örnek Kod

Aşağıdaki örnek, bir e-ticaret sipariş durumunu yöneten gerçekçi bir senaryoyu göstermektedir.

```go
package main

import (
	"fmt"
	"time"
)

// SiparisDurumu, siparişin mevcut durumunu temsil eder.
type SiparisDurumu int

const (
	Beklemede SiparisDurumu = iota
	Hazirlaniyor
	Kargoda
	TeslimEdildi
	IptalEdildi
)

// DurumMesaji, sipariş durumuna göre kullanıcıya gösterilecek mesajı döner.
func DurumMesaji(durum SiparisDurumu) string {
	// Değer eşleştirmeli switch kullanımı
	switch durum {
	case Beklemede:
		return "Siparişiniz alındı, onay bekleniyor."
	case Hazirlaniyor:
		return "Siparişiniz hazırlanıyor."
	case Kargoda:
		return "Siparişiniz kargoya verildi."
	case TeslimEdildi:
		return "Siparişiniz teslim edildi."
	case IptalEdildi:
		return "Siparişiniz iptal edildi."
	default:
		return "Bilinmeyen durum."
	}
}

// KargoSuresi, sipariş durumuna göre tahmini kargo süresini hesaplar.
// Koşullu switch (ifadesiz switch) kullanımını gösterir.
func KargoSuresi(durum SiparisDurumu) time.Duration {
	switch {
	case durum == Kargoda:
		return 2 * 24 * time.Hour // 2 gün
	case durum == Hazirlaniyor:
		return 1 * 24 * time.Hour // 1 gün
	case durum == TeslimEdildi || durum == IptalEdildi:
		return 0 // Süre yok
	default:
		return 3 * 24 * time.Hour // 3 gün
	}
}

// DegerTipi, interface değerinin türünü belirlemek için tür switch kullanır.
func DegerTipi(x interface{}) {
	switch v := x.(type) {
	case int:
		fmt.Printf("Tam sayı: %d\n", v)
	case string:
		fmt.Printf("Metin: %s\n", v)
	case bool:
		fmt.Printf("Boolean: %t\n", v)
	default:
		fmt.Printf("Bilinmeyen tür: %T\n", v)
	}
}

func main() {
	siparis := Kargoda

	// Değer eşleştirmeli switch örneği
	fmt.Println("Durum:", DurumMesaji(siparis))

	// Koşullu switch örneği
	sure := KargoSuresi(siparis)
	fmt.Printf("Tahmini kargo süresi: %.0f gün\n", sure.Hours()/24)

	// Tür switch örneği
	DegerTipi(42)
	DegerTipi("merhaba")
	DegerTipi(true)

	// Fallthrough örneği: Durum kodları sırasıyla birikimli işlenir.
	seviye := 2
	fmt.Print("Seviye açıklaması: ")
	switch seviye {
	case 1:
		fmt.Print("Başlangıç ")
		fallthrough
	case 2:
		fmt.Print("Orta ")
		fallthrough
	case 3:
		fmt.Print("İleri ")
	default:
		fmt.Println("seviye")
	}
	// Çıktı: Seviye açıklaması: Orta İleri seviye
}
```

**Çıktı:**

```
Durum: Siparişiniz kargoya verildi.
Tahmini kargo süresi: 2 gün
Tam sayı: 42
Metin: merhaba
Boolean: true
Seviye açıklaması: Orta İleri seviye
```

## En İyi Uygulamalar

- **Çoklu değerleri virgülle birleştirin.** Aynı işlemi yapan birden fazla `case` için `case 1, 2, 3:` yazın; `fallthrough` kullanmaktan kaçının .
- **İfadesiz `switch`'i `if-else` zincirleri için tercih edin.** Koşullu mantık içeren uzun `if-else` yapıları yerine ifadesiz `switch` daha okunabilirdir .
- **Tür switch'te özel türleri genel türlerden önce yazın.** Bir somut tür, bir interface'i implemente ediyorsa, interface `case`'i daha genel olduğu için önce yazılırsa somut tür `case`'ine asla ulaşılamaz .
- **`default` bloğunu bilinçli kullanın.** `default` zorunlu değildir; ancak beklenmeyen değerleri yakalamak veya hata durumunu ele almak için faydalıdır.

> **Sık Yapılan Hatalar ve Uyarılar**
>
> Go'da her `case` sonunda `break` yazmanıza gerek yoktur; `break` yazmak gereksizdir ve statik analiz araçları tarafından uyarı olarak işaretlenebilir . `fallthrough` anahtar kelimesi **koşulsuz** olarak bir sonraki `case` bloğuna geçer; bir sonraki `case`'in koşulunu kontrol etmez. Ayrıca `fallthrough`, `case` bloğunun son ifadesi olmalıdır; bir `if` bloğunun içinde kullanılamaz . Tür switch'te `fallthrough` **kullanılamaz** . Bir döngü içindeki `switch` bloğunda `break` kullanmak yalnızca `switch`'ten çıkar, döngüden çıkmaz; döngüden çıkmak için etiketli `break` kullanmalısınız.

## Kaynakça

- Go Resmi Dokümantasyonu – Switch Statements: https://go.dev/ref/spec#Switch_statements
- Go Wiki – Switch: https://go.dev/wiki/Switch
- Effective Go – Switch: https://go.dev/doc/effective_go#switch

## Sonuç

Go'da `switch` deyimi, çoklu koşul dallanmasını temiz ve öz bir şekilde ifade etmenin güçlü bir yoludur. Otomatik `break` davranışı sayesinde C benzeri dillerdeki yaygın hatalar önlenir. Değer eşleştirme, koşullu dallanma ve tür ayrımı gibi farklı kullanım biçimleriyle geniş bir yelpazeye hitap eder. `fallthrough` dikkatli kullanılmalı, tür switch'te ise `case` sıralaması doğru yapılmalıdır. Doğru kullanıldığında `switch`, kodun okunabilirliğini ve bakım kolaylığını önemli ölçüde artırır.
