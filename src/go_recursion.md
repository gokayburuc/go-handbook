# GO - Recursion

## Recursion (Özyineleme) Nedir?

Go dilinde özyineleme, bir fonksiyonun **kendini çağırarak** bir problemi daha küçük alt problemlere bölerek çözmesidir . Her özyinelemeli fonksiyonun iki temel bileşeni vardır: problemi durduran **temel durum (base case)** ve problemi küçülterek kendini çağıran **özyinelemeli durum (recursive case)** .

Go özyinelemeyi destekler, ancak **kuyruk özyineleme optimizasyonu (tail call optimization)** yapmaz; bu nedenle derin özyinelemelerde yığın taşması (stack overflow) riski vardır .

## Sözdizimi (Syntax)

Özyinelemeli bir fonksiyon, normal bir fonksiyon gibi tanımlanır; farkı, gövdesi içinde kendini çağırmasıdır.

```go
func fonksiyonAdi(parametreler) donusTipi {
    // Temel durum: özyinelemeyi durdur
    if durum {
        return deger
    }
    // Özyinelemeli durum: kendini çağır
    return fonksiyonAdi(kucultulmusParametreler)
}
```

Anonim fonksiyonlar da özyinelemeli olabilir, ancak fonksiyonun kendini çağırabilmesi için önce `var` ile bir değişkene atanması gerekir .

## Kullanım Alanları

Özyineleme özellikle şu durumlarda tercih edilir :

- **Matematiksel hesaplamalar**: Faktöriyel, Fibonacci dizisi, üs alma gibi tanımı gereği özyinelemeli olan işlemler.
- **Hiyerarşik veri yapıları**: Ağaç (tree) gezintisi, dosya sistemi dizinlerinin taranması.
- **Böl ve yönet algoritmaları**: Quicksort, merge sort gibi problemleri alt parçalara ayıran algoritmalar.
- **Geri izleme (backtracking)**: Labirent çözme, permütasyon üretme gibi deneme-yanılma gerektiren problemler.

## Kullanım Türleri

**1. Temel Özyineleme (Faktöriyel)**

En klasik örnektir. `n == 0` veya `n == 1` temel durumdur; her çağrıda `n` bir azalır .

```go
func faktoriyel(n int) int {
    if n <= 1 {
        return 1 // temel durum
    }
    return n * faktoriyel(n-1) // özyinelemeli durum
}
```

**2. Çift Özyinelemeli Çağrı (Fibonacci)**

Her çağrıda iki kez kendini çağırır. Bu yapı, aynı alt problemlerin tekrar tekrar hesaplanmasına neden olur ve verimsizdir .

```go
func fibonacci(n int) int {
    if n < 2 {
        return n
    }
    return fibonacci(n-1) + fibonacci(n-2)
}
```

**3. Kuyruk Özyineleme (Tail Recursion)**

Özyinelemeli çağrı, fonksiyonun **son işlemi** olduğunda kuyruk özyineleme denir. Bir biriktirici (accumulator) parametre kullanılır. Go bunu otomatik optimize etmez, ancak kodun okunabilirliği için tercih edilebilir .

```go
func kuyrukFaktoriyel(n int) int {
    return yardimci(n, 1)
}

func yardimci(n, biriktirici int) int {
    if n == 0 {
        return biriktirici
    }
    return yardimci(n-1, n*biriktirici) // son işlem özyinelemeli çağrı
}
```

**4. Ağaç Gezintisi**

Özyineleme, ağaç benzeri yapılar için doğal bir çözümdür. Her düğüm, çocuklarını özyinelemeli olarak işler .

```go
type Dugum struct {
    Deger    int
    Cocuklar []*Dugum
}

func topla(d *Dugum) int {
    if d == nil {
        return 0
    }
    toplam := d.Deger
    for _, c := range d.Cocuklar {
        toplam += topla(c) // her çocuk için özyinele
    }
    return toplam
}
```

## Örnek Kod

Aşağıdaki örnek, bir dosya sistemi dizinini özyinelemeli olarak tarayan ve toplam dosya boyutunu hesaplayan gerçekçi bir senaryoyu göstermektedir.

```go
package main

import (
	"fmt"
	"os"
	"path/filepath"
)

// dosyaBoyutu, verilen yolun bir dosya mı yoksa dizin mi olduğunu kontrol eder.
// Dizin ise, içindeki her öğe için kendini özyinelemeli olarak çağırır.
func dosyaBoyutu(yol string) (int64, error) {
	// Yol hakkında bilgi al
	bilgi, err := os.Stat(yol)
	if err != nil {
		return 0, err // temel durum: erişilemeyen yol
	}

	// Temel durum: bu bir dosya ise boyutunu döndür
	if !bilgi.IsDir() {
		return bilgi.Size(), nil
	}

	// Özyinelemeli durum: dizin ise içindekileri topla
	var toplam int64
	girdiler, err := os.ReadDir(yol)
	if err != nil {
		return 0, err
	}

	for _, girdi := range girdiler {
		altYol := filepath.Join(yol, girdi.Name())
		boyut, err := dosyaBoyutu(altYol) // özyinelemeli çağrı
		if err != nil {
			fmt.Printf("Uyarı: %s okunamadı: %v\n", altYol, err)
			continue
		}
		toplam += boyut
	}
	return toplam, nil
}

// fibonacciHesapla, memoization (önbellekleme) ile Fibonacci hesaplar.
// Aynı alt problemlerin tekrar hesaplanmasını önler.
func fibonacciHesapla(n int, memo map[int]int) int {
	if n < 2 {
		return n // temel durum
	}
	// Önbellekte varsa doğrudan döndür
	if deger, varMi := memo[n]; varMi {
		return deger
	}
	// Hesapla ve önbelleğe kaydet
	memo[n] = fibonacciHesapla(n-1, memo) + fibonacciHesapla(n-2, memo)
	return memo[n]
}

func main() {
	// 1. Basit faktöriyel
	fmt.Printf("5! = %d\n", faktoriyel(5))

	// 2. Memoization ile Fibonacci
	memo := make(map[int]int)
	fmt.Printf("Fibonacci(10) = %d\n", fibonacciHesapla(10, memo))

	// 3. Dizin boyutu hesaplama (özyinelemeli dosya tarama)
	// Mevcut dizini tarar; hata olursa atlar.
	if boyut, err := dosyaBoyutu("."); err == nil {
		fmt.Printf("Mevcut dizin boyutu: %d bayt\n", boyut)
	}
}
```

**Çıktı (örnek):**

```
5! = 120
Fibonacci(10) = 55
Mevcut dizin boyutu: 4096 bayt
```

## En İyi Uygulamalar

- **Her zaman bir temel durum tanımlayın.** Temel durum olmadan fonksiyon sonsuz döngüye girer ve sonunda yığın taşması ile çöker .
- **Her çağrıda problemi temel duruma yaklaştırın.** Parametrenin her adımda küçülmesi veya azalması gerekir; aksi halde özyineleme sonlanmaz .
- **Tekrarlı hesaplamalar için memoization kullanın.** Fibonacci gibi çift özyinelemeli çağrılarda, hesaplanan sonuçları bir `map`'te saklayarak üstel zaman karmaşıklığını doğrusala indirebilirsiniz .
- **Derin özyinelemelerde dikkatli olun.** Go kuyruk özyinelemeyi optimize etmez; kullanıcı girdisine bağlı derinliklerde maksimum derinlik sınırı koyun veya döngü kullanın .
- **Büyük sayılarda taşma riskini göz önünde bulundurun.** `int` türü faktöriyel gibi hızlı büyüyen sonuçlarda taşabilir; gerekirse `math/big` paketini kullanın .

> **Sık Yapılan Hatalar ve Uyarılar**
>
> **Temel durumun eksikliği** en yaygın hatadır; fonksiyon kendini sonsuza kadar çağırır ve `runtime: goroutine stack exceeds ... fatal error: stack overflow` hatasıyla program çöker . **Parametrenin temel duruma yaklaşmaması** da aynı sonucu doğurur; örneğin negatif bir sayıda `n-1` yapmak değeri daha da küçültür ve temel duruma asla ulaşılamaz . **Go'nun kuyruk özyinelemeyi optimize etmediğini unutmayın**; `return faktoriyel(n-1)` yazmak bile yığın kullanımını azaltmaz . **Döngü ile çözülebilecek problemlerde özyineleme aşırı bellek tüketebilir**; her çağrı yeni bir yığın çerçevesi oluşturur .

## Kaynakça

- Go by Example – Recursion: https://gobyexample.com/recursion
- Go Packages – Recursion (go-dsa): https://pkg.go.dev/github.com/spring1843/go-dsa@v0.0.2/recursion
- Go Turu – Fonksiyonlar: https://go.dev/tour/basics/4

## Sonuç

Go'da özyineleme, bir fonksiyonun kendini çağırarak karmaşık problemleri daha basit alt problemlere böldüğü güçlü bir tekniktir. Temel durum ve özyinelemeli durum olmak üzere iki bileşeni vardır; temel durumun eksikliği yığın taşmasına yol açar. Faktöriyel ve Fibonacci gibi matematiksel problemlerde, ağaç gezintisinde ve dosya sistemi taramasında doğal bir çözüm sunar. Ancak Go kuyruk özyinelemeyi optimize etmediği için derin özyinelemelerde dikkatli olunmalı, tekrarlı hesaplamalarda memoization kullanılmalı ve gerektiğinde döngü tabanlı alternatifler değerlendirilmelidir.
