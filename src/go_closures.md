# GO - Closures

## Closures Nedir?

Go dilinde closure, **kendi gövdesi dışındaki değişkenlere başvuran bir fonksiyon değeridir** . Bu fonksiyon, başvurduğu değişkenlere erişebilir ve onlara değer atayabilir; bu anlamda fonksiyon o değişkenlere "bağlıdır" . Go'da fonksiyon literalleri (anonim fonksiyonlar) closure oluşturur; dış fonksiyondaki değişkenler, erişilebilir oldukları sürece hayatta kalır .

Closure'lar, değişkenlerin kapsamını sınırlamaya yardımcı olur. Closure olmadan, birden fazla fonksiyonun aynı değişkene erişmesi için değişkenin paket kapsamında (package scope) tanımlanması gerekirdi . Bu da gereksiz global duruma ve istenmeyen yan etkilere yol açabilir.

## Sözdizimi (Syntax)

Closure, bir fonksiyon literal'inin bir değişkene atanması veya başka bir fonksiyondan döndürülmesiyle oluşturulur.

```go
// Bir değişkene atanan closure
toplam := 0
ekle := func(x int) int {
    toplam += x // dış değişkeni yakalar
    return toplam
}

// Başka bir fonksiyondan döndürülen closure
func sayac() func() int {
    i := 0
    return func() int {
        i++
        return i
    }
}
```

`sayac` fonksiyonu her çağrıldığında, döndürülen closure kendi `i` değişkenine bağlıdır. Her yeni closure, bağımsız bir `i` değerine sahiptir .

## Kullanım Alanları

Closure'lar özellikle şu durumlarda tercih edilir :

- **Durum saklama (State encapsulation)**: Bir fonksiyonun çağrılar arasında durumu koruması gerektiğinde (sayaç, biriktirici).
- **Fabrika fonksiyonları**: Özelleştirilmiş davranışa sahip fonksiyonlar üretmek (örneğin, belirli bir sonek ekleyen fonksiyon).
- **Geri çağırma (callback) ve ara katman (middleware)**: HTTP işleyicilerinde veya olay dinleyicilerinde kullanılır.
- **Kısmi uygulama (partial application)**: Bir fonksiyonun bazı parametrelerini sabitleyip yeni bir fonksiyon üretmek.

## Kullanım Türleri

**1. Anonim Fonksiyonun Değişkene Atanması**

En temel closure kullanımıdır. Anonim fonksiyon, dış kapsamdaki bir değişkeni yakalar .

```go
a := 0
adder := func(b int) int {
    a += b
    return a
}
fmt.Println(adder(2)) // 2
fmt.Println(adder(3)) // 5
fmt.Println(a)        // 5
```

Burada `a` fonksiyona argüman olarak geçmediği halde değişmektedir; çünkü closure `a`'yı referans olarak yakalar .

**2. Fonksiyondan Fonksiyon Döndürme**

Fabrika fonksiyonu kalıbıdır. Döndürülen her closure kendi bağımsız durumuna sahiptir .

```go
func intSeq() func() int {
    i := 0
    return func() int {
        i++
        return i
    }
}

nextInt := intSeq()
fmt.Println(nextInt()) // 1
fmt.Println(nextInt()) // 2
newInts := intSeq()
fmt.Println(newInts()) // 1 (bağımsız sayaç)
```

**3. Goroutine ile Closure Kullanımı**

Closure'lar `go` anahtar kelimesiyle birlikte kullanılabilir; ancak döngü değişkenlerinin yakalanması konusunda dikkatli olunmalıdır .

```go
// YANLIŞ: tüm goroutine'ler aynı val değişkenini paylaşır
for _, val := range values {
    go func() {
        fmt.Println(val) // çoğu zaman son değer yazdırılır
    }()
}

// DOĞRU: val parametre olarak geçirilir
for _, val := range values {
    go func(v interface{}) {
        fmt.Println(v)
    }(val)
}
```

## Örnek Kod

Aşağıdaki örnek, bir e-ticaret sisteminde alışveriş sepeti toplamını hesaplayan ve indirim uygulayan closure tabanlı bir yapıyı göstermektedir.

```go
package main

import "fmt"

// sepetOlustur, başlangıç bakiyesi olan bir sepet closure'ı döndürür.
// Döndürülen fonksiyon, ürün ekleme ve toplam görüntüleme işlemlerini yönetir.
func sepetOlustur() (ekle func(float64) float64, toplam func() float64) {
	bakiye := 0.0

	ekle = func(fiyat float64) float64 {
		bakiye += fiyat
		return bakiye
	}

	toplam = func() float64 {
		return bakiye
	}

	return ekle, toplam
}

// indirimUygula, belirli bir indirim oranını uygulayan bir fonksiyon döndürür.
// Fabrika fonksiyonu kalıbını gösterir.
func indirimUygula(oran float64) func(float64) float64 {
	return func(tutar float64) float64 {
		return tutar * (1 - oran)
	}
}

func main() {
	// Birinci sepet: closure ile durum saklama
	urunEkle, sepetToplam := sepetOlustur()
	urunEkle(150.0)
	urunEkle(75.50)
	urunEkle(300.0)
	fmt.Printf("Sepet toplamı: %.2f TL\n", sepetToplam())

	// İkinci sepet: bağımsız durum
	baskaEkle, baskaToplam := sepetOlustur()
	baskaEkle(50.0)
	fmt.Printf("Diğer sepet toplamı: %.2f TL\n", baskaToplam())

	// İndirim closure'ı: %20 indirim uygula
	yuzde20 := indirimUygula(0.20)
	fmt.Printf("İndirimli tutar: %.2f TL\n", yuzde20(sepetToplam()))

	// İndirim closure'ı: %50 indirim uygula
	yuzde50 := indirimUygula(0.50)
	fmt.Printf("Yarı fiyat: %.2f TL\n", yuzde50(sepetToplam()))

	// Döngüde goroutine ile closure kullanımı (doğru yöntem)
	fmt.Println("\nGoroutine çıktıları:")
	for i := 1; i <= 3; i++ {
		go func(id int) {
			fmt.Printf("  Goroutine %d çalıştı\n", id)
		}(i) // i parametre olarak geçiriliyor
	}

	// Goroutine'lerin bitmesini bekle (basit örnek için)
	var girdi string
	fmt.Scanln(&girdi)
}
```

**Çıktı:**

```
Sepet toplamı: 525.50 TL
Diğer sepet toplamı: 50.00 TL
İndirimli tutar: 420.40 TL
Yarı fiyat: 262.75 TL

Goroutine çıktıları:
  Goroutine 1 çalıştı
  Goroutine 2 çalıştı
  Goroutine 3 çalıştı
```

## En İyi Uygulamalar

- **Döngü değişkenlerini goroutine closure'larına parametre olarak geçirin.** Bu, her goroutine'in döngünün o anki değerini almasını garanti eder .
- **Closure'ın davranışını belgeleyin.** Hangi dış değişkenleri yakaladığını ve bu değişkenlerin yaşam döngüsünü açıklayan yorumlar ekleyin .
- **Bellek etkilerini göz önünde bulundurun.** Yakalanan değişkenler, closure erişilebilir olduğu sürece bellekte kalır; büyük veri yapılarında bu durum bellek sızıntısına yol açabilir .
- **Durum saklamak için closure kullanın, global değişkenden kaçının.** Closure, değişkenin kapsamını daraltarak istenmeyen erişimi önler .

> **Sık Yapılan Hatalar ve Uyarılar**
>
> **Döngüde goroutine ile closure kullanırken değişkeni paylaşmak** en yaygın hatadır. `for val := range values { go func() { fmt.Println(val) }() }` yazıldığında, tüm goroutine'ler aynı `val` değişkenine bağlanır ve çoğu zaman son değeri yazdırır . Çözüm, `val`'ı closure'a parametre olarak geçirmektir: `go func(v interface{}) { ... }(val)`. **Döngü gövdesinde tanımlanan değişkenler** ise her iterasyonda ayrı ayrı oluşturulduğu için güvenle yakalanabilir . Ayrıca, closure'ların yakaladığı değişkenler referans olarak tutulduğu için, döngü değişkeninin adresi değişmez; bu nedenle goroutine'lerin ne zaman çalışacağı belirsizse, parametre geçirme yöntemi her zaman tercih edilmelidir.

## Kaynakça

- Go Turu – Function Closures: https://go.dev/tour/moretypes/25
- Go Wiki – Common Mistakes (Closures with Goroutines): https://go.dev/wiki/CommonMistakes
- Effective Go – Function literals: https://go.dev/doc/effective_go#function_literals

## Sonuç

Go'da closure'lar, bir fonksiyonun kendi gövdesi dışındaki değişkenleri yakalayıp çağrılar arasında durumu korumasını sağlayan güçlü bir yapıdır. Anonim fonksiyonlar aracılığıyla oluşturulur ve fabrika fonksiyonları, durum saklama ve geri çağırma gibi kalıplarda yaygın olarak kullanılır. En kritik nokta, goroutine'lerle birlikte kullanıldığında döngü değişkenlerinin yakalanmasıdır; bu durumda değişkeni parametre olarak geçirmek yarış koşullarını önler. Doğru kullanıldığında closure'lar, kodun kapsamını daraltır ve global duruma olan ihtiyacı azaltır.
