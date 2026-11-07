# Go'da Generics (Tip Parametreleri)

## Generics Nedir?

Go dilinde generics, Go 1.18 sürümüyle dile eklenen ve fonksiyonlar ile veri yapılarının **belirli bir türe bağlı kalmadan** birden fazla türle çalışabilmesini sağlayan bir özelliktir . Tip parametreleri (type parameters) aracılığıyla, aynı mantığı farklı türler için tekrar tekrar yazmak yerine tek bir genel tanım oluşturulur. Bu, kod tekrarını azaltırken Go'nun güçlü tip güvenliğini korur .

Generics, özellikle bağlantılı liste, yığın (stack), önbellek (cache) gibi veri yapılarında ve `filter`, `map`, `reduce` gibi dizi işlemlerinde kodun sadeleşmesini sağlar . Ancak Go'nun generics desteği, arayüzlerin (interfaces) yerini almak için değil, onları tamamlamak için tasarlanmıştır .

## Sözdizimi (Syntax)

Tip parametreleri, fonksiyon veya tür adından sonra **köşeli parantez** içinde tanımlanır:

```go
func FonksiyonAdi[T any](parametre T) T {
    // T, herhangi bir tür olabilir
    return parametre
}
```

`T` bir tip parametresidir; `any` ise bu parametrenin alabileceği türleri kısıtlayan bir **constraint** (kısıt)'tir. `any`, her türün kabul edildiği anlamına gelir .

Generic bir tür tanımlamak için:

```go
type Yigin[T any] struct {
    elemanlar []T
}
```

## Kullanım Alanları

- **Generic veri yapıları**: Farklı türleri tutabilen yığın, kuyruk, bağlantılı liste, önbellek gibi yapılar .
- **Dizi/map yardımcı fonksiyonları**: `Filter`, `Map`, `Contains`, `Index` gibi her türle çalışabilen fonksiyonlar .
- **Kod tekrarını azaltma**: Aynı mantığın `int`, `string`, `float64` gibi birden fazla tür için yazılması gerektiğinde .
- **Tip güvenliği ile `any` kullanımını azaltma**: Çalışma zamanı tip dönüşümü (type assertion) yerine derleme zamanı tip kontrolü sağlar .

## Kullanım Türleri

**1. Generic Fonksiyon**

```go
func EsitMi[T comparable](a, b T) bool {
    return a == b
}
```

`comparable` kısıtı, `==` ve `!=` operatörlerinin kullanılabileceği türleri (sayılar, string, bool, pointer vb.) kabul eder .

**2. Generic Tür (Struct)**

```go
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}
```

**3. Kısıt (Constraint) Tanımlama**

Kısıtlar, bir tip parametresinin hangi türleri alabileceğini sınırlar. `~` operatörü, temel türü (underlying type) belirtilen türleri kapsar:

```go
type Sayisal interface {
    ~int | ~float64
}

func Topla[T Sayisal](a, b T) T {
    return a + b
}
```

`~int`, `int` türünün yanı sıra `type MyInt int` gibi temel türü `int` olan tüm tanımlı türleri de kapsar .

**4. Tip Çıkarımı (Type Inference)**

Go derleyicisi, çoğu durumda tip parametrelerini argümanlardan otomatik olarak çıkarır:

```go
// Açık tip belirtimi
EsitMi[int](5, 5)

// Tip çıkarımı (önerilen)
EsitMi(5, 5)
```

## Örnek Kod

Aşağıdaki örnek, generic bir önbellek (cache) yapısını ve generic `Filter` fonksiyonunu göstermektedir.

```go
package main

import (
	"fmt"
	"sync"
)

// Cache, herhangi bir anahtar ve değer türüyle çalışan generic bir önbellektir.
// K: anahtar türü (comparable olmalı, map anahtarı olarak kullanılabilmesi için)
// V: değer türü (herhangi bir tür olabilir)
type Cache[K comparable, V any] struct {
	mu    sync.RWMutex
	items map[K]V
}

// YeniCache, boş bir önbellek oluşturur ve döndürür.
func YeniCache[K comparable, V any]() *Cache[K, V] {
	return &Cache[K, V]{items: make(map[K]V)}
}

// Koy, önbelleğe bir anahtar-değer çifti ekler veya günceller.
func (c *Cache[K, V]) Koy(anahtar K, deger V) {
	c.mu.Lock()
	defer c.mu.Unlock()
	c.items[anahtar] = deger
}

// Al, anahtara karşılık gelen değeri döndürür.
// İkinci dönüş değeri, anahtarın önbellekte olup olmadığını belirtir.
func (c *Cache[K, V]) Al(anahtar K) (V, bool) {
	c.mu.RLock()
	defer c.mu.RUnlock()
	deger, varMi := c.items[anahtar]
	return deger, varMi
}

// Filtrele, bir dilimdeki koşulu sağlayan elemanları yeni bir dilime toplar.
// T: dilim elemanlarının türü (herhangi bir tür)
func Filtrele[T any](dilim []T, kosul func(T) bool) []T {
	var sonuc []T
	for _, eleman := range dilim {
		if kosul(eleman) {
			sonuc = append(sonuc, eleman)
		}
	}
	return sonuc
}

// Sayisal, toplama işlemini destekleyen türleri tanımlayan bir kısıttır.
type Sayisal interface {
	~int | ~float64
}

// Topla, bir dilimdeki tüm sayıları toplar.
func Topla[T Sayisal](sayilar []T) T {
	var toplam T
	for _, s := range sayilar {
		toplam += s
	}
	return toplam
}

func main() {
	// 1. Generic önbellek: string anahtar, int değer
	stringCache := YeniCache[string, int]()
	stringCache.Koy("bir", 1)
	stringCache.Koy("iki", 2)

	if deger, varMi := stringCache.Al("iki"); varMi {
		fmt.Printf("Önbellekten okunan: %d\n", deger)
	}

	// 2. Generic önbellek: int anahtar, string değer
	intCache := YeniCache[int, string]()
	intCache.Koy(100, "yüz")
	if deger, _ := intCache.Al(100); deger != "" {
		fmt.Printf("int anahtarlı önbellek: %s\n", deger)
	}

	// 3. Generic Filtrele fonksiyonu
	sayilar := []int{1, 2, 3, 4, 5, 6}
	ciftler := Filtrele(sayilar, func(n int) bool { return n%2 == 0 })
	fmt.Printf("Çift sayılar: %v\n", ciftler)

	isimler := []string{"Ali", "Ayşe", "Mehmet", "Zeynep"}
	uzunIsimler := Filtrele(isimler, func(s string) bool { return len(s) > 4 })
	fmt.Printf("Uzun isimler: %v\n", uzunIsimler)

	// 4. Generic Topla fonksiyonu (Sayisal kısıtı ile)
	intToplam := Topla([]int{10, 20, 30})
	floatToplam := Topla([]float64{1.5, 2.5, 3.0})
	fmt.Printf("int toplam: %d\n", intToplam)
	fmt.Printf("float toplam: %.2f\n", floatToplam)

	// 5. Tip çıkarımı: açık tip belirtimi gerekmez
	fmt.Printf("Filtrele (tip çıkarımı): %v\n", Filtrele(sayilar, func(n int) bool { return n > 3 }))
}
```

**Çıktı:**

```
Önbellekten okunan: 2
int anahtarlı önbellek: yüz
Çift sayılar: [2 4 6]
Uzun isimler: [Mehmet Zeynep]
int toplam: 60
float toplam: 7.00
Filtrele (tip çıkarımı): [4 5 6]
```

## En İyi Uygulamalar

- **Önce somut türle başlayın, sonra genelleştirin.** Yalnızca ikinci bir tür ortaya çıktığında generics'e başvurun; tek kullanımlık generic kod gereksiz karmaşıklıktır .
- **Kısıtları mümkün olduğunca dar tutun.** `any` yerine `comparable` veya özel bir arayüz kullanmak, derleyicinin daha fazla hata yakalamasını sağlar .
- **Standart kısıtları tercih edin.** Go 1.21+ ile gelen `cmp.Ordered` ve `slices`/`maps` paketlerindeki hazır fonksiyonları kullanın; kendi kısıtlarınızı yazmak yerine bunları tercih edin .
- **Arayüz yeterliyse generics kullanmayın.** Yalnızca bir metodu çağırmak için tip parametresine ihtiyaç yoksa, doğrudan arayüz kullanın .
- **Anlamlı isimler seçin.** `T` genel tür, `K` anahtar, `V` değer, `E` eleman için yaygın kısaltmalardır .

> **Sık Yapılan Hatalar ve Uyarılar**
>
> **Aşırı geniş kısıt (`any`) kullanmak**: Generic fonksiyon içinde `val.Serialize()` gibi bir metot çağırmak istiyorsanız, `any` kısıtı bu metodu garanti etmez ve derleme hatası verir. Çözüm, `Serialize() string` metodunu içeren bir kısıt tanımlamaktır . **Arayüz yerine generics kullanmak**: Yalnızca bir metodu çağırmak için tip parametresi kullanmak gereksizdir; `func Process[T io.Reader](r T)` yerine doğrudan `func Process(r io.Reader)` yazmak daha okunabilir ve aynı performansı sunar . **Tek kullanımlık generic kod**: Yalnızca `int` ile çağrılan bir `Sum[T constraints.Integer]` fonksiyonu, somut `SumInts([]int)` fonksiyonundan daha karmaşıktır ve fayda sağlamaz . **Go 1.18 öncesi sürümlerle uyumluluk**: Generics, Go 1.18 ve sonrasında kullanılabilir; daha eski sürümlerde derlenmez.

## Kaynakça

- Go Blog – An Introduction To Generics: https://go.dev/blog/intro-generics
- Go by Example – Generics: https://gobyexample.com/generics
- Go Blog – When To Use Generics: https://go.dev/blog/when-generics
- Go Turu – Generics: https://go.dev/tour/generics/1

## Sonuç

Go'da generics, Go 1.18 ile gelen tip parametreleri sayesinde fonksiyonların ve veri yapılarının birden fazla türle tip güvenli bir şekilde çalışmasını sağlar. `func Filtrele[T any](...)` ve `type Cache[K comparable, V any]` gibi tanımlarla kod tekrarı azalır. Kısıtlar (`comparable`, `~int | ~float64`), tip parametrelerinin alabileceği türleri sınırlar. Tip çıkarımı sayesinde çoğu çağrıda açık tip belirtimi gerekmez. Ancak generics her derde deva değildir: arayüzlerin yeterli olduğu durumlarda veya tek kullanımlık kodda generics gereksiz karmaşıklık yaratır. Doğru kullanıldığında, Go kodunu daha kısa, güvenli ve sürdürülebilir hale getirir.
