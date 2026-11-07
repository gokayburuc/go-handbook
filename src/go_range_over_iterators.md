# GO - Range Over Iterators

## Range Over Iterators Nedir?

Go 1.23 sürümüyle birlikte dile **iterator (yineleyici)** desteği eklendi. Bu özellik sayesinde `range` döngüsü, yalnızca dilimler (slice), map'ler ve kanallar üzerinde değil, **hemen hemen her şey** üzerinde çalışabilir hale geldi . Go'da bir iterator, özel imzaya sahip bir fonksiyondur; yani `range` ile kullanılabilecek şekilde tasarlanmış bir fonksiyon değeridir. Bu yapı, veri yapılarının iç detaylarını dışarıya sızdırmadan üzerlerinde gezinmeyi sağlar.

## Sözdizimi (Syntax)

Go'da bir iterator, `iter.Seq[T]` türünde bir fonksiyondur. Bu türün tanımı şu şekildedir:

```go
type Seq[T any] func(yield func(T) bool)
```

Iterator fonksiyonu, kendisine bir **`yield`** fonksiyonu alır. `yield`, iterator'ın ürettiği her eleman için çağrılır ve `bool` döndürür. `yield` `true` döndüğü sürece iterasyon devam eder; `false` döndüğünde iterator erken sonlanır .

```go
func iteratorUreten() iter.Seq[int] {
    return func(yield func(int) bool) {
        for i := 0; i < 3; i++ {
            if !yield(i) {
                return // erken sonlanma
            }
        }
    }
}
```

Kullanımı ise normal bir `range` döngüsüyle aynıdır:

```go
for v := range iteratorUreten() {
    fmt.Println(v)
}
```

## Kullanım Alanları

- **Özel veri yapıları üzerinde gezinme**: Bağlantılı liste, ağaç, grafik gibi yapıların elemanlarına standart `range` sözdizimiyle erişmek .
- **Sonsuz diziler**: Fibonacci sayıları gibi tanımı gereği sonsuz olan dizileri tembel (lazy) olarak üretmek .
- **Bellek verimliliği**: Büyük veri kümelerini önce bir dilime kopyalamadan parça parça işlemek; `strings.SplitSeq` bunun iyi bir örneğidir .
- **Standart kütüphane entegrasyonu**: `slices` ve `maps` paketlerindeki iterator yardımcılarıyla (`slices.Collect`, `slices.All` vb.) uyumlu çalışmak .

## Kullanım Türleri

**1. Yapı Metodu Olarak Iterator**

Bir veri yapısı, `iter.Seq[T]` döndüren bir metot tanımlayarak kendi elemanları üzerinde gezinmeyi sağlar .

```go
func (lst *List[T]) All() iter.Seq[T] {
    return func(yield func(T) bool) {
        for e := lst.head; e != nil; e = e.next {
            if !yield(e.val) {
                return
            }
        }
    }
}
```

**2. Sonsuz Iterator (Lazy Üretim)**

Iterator'ın altında bir veri yapısı olmak zorunda değildir; hatta sonlu olmak zorunda bile değildir. `yield` `true` döndüğü sürece üretim devam eder .

```go
func genFib() iter.Seq[int] {
    return func(yield func(int) bool) {
        a, b := 0, 1
        for {
            if !yield(a) {
                return
            }
            a, b = b, a+b
        }
    }
}
```

**3. Standart Kütüphane Iterator Yardımcıları**

`slices.Collect`, herhangi bir iterator'ın tüm değerlerini bir dilime toplar. `strings.SplitSeq` ise bir string'i önce dilime dönüştürmeden parçaları üzerinde gezinmeyi sağlar .

```go
all := slices.Collect(lst.All())
for part := range strings.SplitSeq("go-by-example", "-") {
    fmt.Println(part)
}
```

**4. Erken Sonlanma**

Döngüde `break` veya erken `return` kullanıldığında, iterator'a geçirilen `yield` fonksiyonu `false` döndürür ve iterator üretimi durdurur .

```go
for n := range genFib() {
    if n >= 10 {
        break // yield false döner, iterator durur
    }
    fmt.Println(n)
}
```

## Örnek Kod

Aşağıdaki örnek, generic bir bağlantılı liste üzerinde iterator kullanımını ve sonsuz Fibonacci üretimini göstermektedir.

```go
package main

import (
	"fmt"
	"iter"
	"slices"
)

// List, generic bir bağlantılı liste türüdür.
type List[T any] struct {
	head, tail *element[T]
}

// element, listenin bir düğümünü temsil eder.
type element[T any] struct {
	next *element[T]
	val  T
}

// Push, listenin sonuna yeni bir eleman ekler.
func (lst *List[T]) Push(v T) {
	if lst.tail == nil {
		lst.head = &element[T]{val: v}
		lst.tail = lst.head
	} else {
		lst.tail.next = &element[T]{val: v}
		lst.tail = lst.tail.next
	}
}

// All, liste elemanları üzerinde gezinmeyi sağlayan bir iterator döndürür.
// iter.Seq[T], range ile kullanılabilen fonksiyon türüdür.
func (lst *List[T]) All() iter.Seq[T] {
	return func(yield func(T) bool) {
		for e := lst.head; e != nil; e = e.next {
			// yield false dönerse (break/return) iterasyonu durdur
			if !yield(e.val) {
				return
			}
		}
	}
}

// genFib, sonsuz Fibonacci dizisi üreten bir iterator döndürür.
// yield true döndüğü sürece üretim devam eder.
func genFib() iter.Seq[int] {
	return func(yield func(int) bool) {
		a, b := 0, 1
		for {
			if !yield(a) {
				return
			}
			a, b = b, a+b
		}
	}
}

func main() {
	lst := List[int]{}
	lst.Push(10)
	lst.Push(13)
	lst.Push(23)

	// 1. Iterator üzerinde normal range döngüsü
	for e := range lst.All() {
		fmt.Println(e)
	}

	// 2. slices.Collect ile iterator değerlerini dilime toplama
	all := slices.Collect(lst.All())
	fmt.Println("all:", all)

	// 3. Erken sonlanma: 10'a ulaşınca döngüden çık
	for n := range genFib() {
		if n >= 10 {
			break
		}
		fmt.Println(n)
	}
}
```

**Çıktı:**

```
10
13
23
all: [10 13 23]
0
1
1
2
3
5
8
```

## En İyi Uygulamalar

- **`yield` dönüş değerini mutlaka kontrol edin.** `if !yield(v) { return }` kalıbı, çağıran tarafın `break` veya erken `return` ile iterasyonu durdurmasına olanak tanır; bu kontrol atlanırsa kaynaklar boşa tüketilir .
- **Iterator'ı tembel (lazy) tutun.** Tüm elemanları önce bir dilime toplayıp sonra döndürmek yerine, `yield` ile parça parça üretin; bu bellek kullanımını azaltır .
- **Sonsuz iterator'larda dikkatli olun.** `genFib` gibi sonsuz üreten iterator'lar, çağıran tarafta mutlaka bir sonlandırma koşulu (`break`) gerektirir .
- **Standart yardımcıları tercih edin.** `slices.Collect`, `slices.All` ve `strings.SplitSeq` gibi hazır fonksiyonlar, elle yazılan döngülerden daha okunabilir ve bakımı kolaydır .

> **Sık Yapılan Hatalar ve Uyarılar**
>
> **`yield` dönüş değerini yok saymak**, çağıran taraf `break` yaptığında iterator'ın üretime devam etmesine ve gereksiz işlem yapmasına yol açar; ayrıca sonsuz iterator'larda bu durum döngünün hiç bitmemesine neden olabilir . **Iterator içinde paylaşılan durumu değiştirmek**, aynı iterator'ın birden fazla yerde kullanılması durumunda beklenmedik sonuçlara yol açabilir; iterator'lar mümkün olduğunca durumsuz veya izole durumlu olmalıdır. **Go 1.23 öncesi sürümlerle uyumluluk**: `iter` paketi ve `range` üzerinde iterator desteği Go 1.23 ve sonrasında kullanılabilir; daha eski sürümlerde derlenmez .

## Kaynakça

- Go by Example – Range over Iterators: https://gobyexample.com/range-over-iterators
- Go Paket Dokümantasyonu – `iter`: https://pkg.go.dev/iter
- Go Paket Dokümantasyonu – `slices`: https://pkg.go.dev/slices
- Go Paket Dokümantasyonu – `strings` (`SplitSeq`): https://pkg.go.dev/strings#SplitSeq

## Sonuç

Go 1.23 ile gelen iterator desteği, `range` döngüsünün yalnızca yerleşik veri yapılarıyla sınırlı kalmamasını sağlar. `iter.Seq[T]` türündeki fonksiyonlar, `yield` aracılığıyla eleman üretir ve çağıran tarafın erken sonlanma isteğine saygı duyar. Bu yapı sayesinde bağlantılı listeler, ağaçlar ve hatta sonsuz diziler standart `range` sözdizimiyle gezilebilir. `slices` ve `strings` paketlerindeki iterator yardımcıları, bellek verimliliği ve okunabilirlik sunar. `yield` dönüş değerini kontrol etmek ve sonsuz iterator'larda sonlandırma koşulu bulundurmak, bu özelliğin güvenli kullanımının temelini oluşturur.
