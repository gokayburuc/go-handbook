# GO - Goroutines

## Goroutine Nedir?

Go dilinde goroutine, Go çalışma zamanı (runtime) tarafından yönetilen **hafif bir iş parçacığıdır** (thread) . İşletim sistemi thread'lerinden çok daha hafiftir; tek bir OS thread'i birden fazla goroutine'i yönetebilir, bu nedenle ilişki 1:1 değildir . Goroutine'ler sayesinde Go'da eşzamanlı (concurrent) programlar yazmak son derece kolaydır: bir fonksiyon çağrısının önüne `go` anahtar kelimesini eklemek yeterlidir.

Go'nun eşzamanlılık felsefesi şu sloganda özetlenir: **"Hafızayı paylaşarak iletişim kurmayın; iletişim kurarak hafızayı paylaşın."** Bu yaklaşım, kanallar (channels) aracılığıyla veri alışverişini teşvik eder ve yarış koşullarını (race conditions) azaltır .

## Sözdizimi (Syntax)

Bir goroutine başlatmak için fonksiyon çağrısının önüne `go` anahtar kelimesi eklenir:

```go
go fonksiyonAdi(parametreler)
```

Anonim fonksiyonlar da goroutine olarak çalıştırılabilir:

```go
go func() {
    fmt.Println("Merhaba goroutine")
}()
```

`go` ifadesi çalıştırıldığında, fonksiyonun çalıştırılması sorumluluğu Go runtime'ın zamanlayıcısına (scheduler) devredilir. Çalıştırma sırası garanti edilmez ve kontrol edilemez .

## Kullanım Alanları

- **Arka plan işlemleri**: Kullanıcı arayüzünü veya ana akışı bloklamadan veri işleme, dosya tarama gibi işleri yürütmek .
- **Paralel hesaplama**: Bir işi birden fazla parçaya bölerek aynı anda işlemek (örneğin, bir dilimin elemanlarını toplama) .
- **Eşzamanlı istekler**: HTTP istekleri, veritabanı sorguları gibi G/Ç (I/O) yoğun işlemleri paralelleştirmek.
- **Worker pool (İşçi havuzu)**: Sabit sayıda goroutine ile bir kanaldan iş alıp işlemek .

## Kullanım Türleri

**1. Basit Goroutine Başlatma**

```go
func selamla(isim string) {
    fmt.Println("Merhaba", isim)
}

func main() {
    go selamla("Ayşe") // ayrı bir goroutine'de çalışır
    selamla("Mehmet")   // ana goroutine'de çalışır
}
```

**2. Anonim Fonksiyon ile Goroutine**

```go
go func(mesaj string) {
    fmt.Println(mesaj)
}("anonim")
```

**3. `sync.WaitGroup` ile Senkronizasyon**

`time.Sleep` ile beklemek güvenilir değildir. `sync.WaitGroup`, goroutine'lerin bitmesini beklemek için kullanılır .

```go
var wg sync.WaitGroup
wg.Add(1) // sayaç artır

go func() {
    defer wg.Done() // bitince sayacı azalt
    fmt.Println("iş bitti")
}()

wg.Wait() // sayaç sıfırlanana kadar bekle
```

**4. Kanallar (Channels) ile İletişim**

Goroutine'ler arasında veri paylaşımı için kanallar kullanılır. Kanallar, açık kilitler olmadan senkronizasyon sağlar .

```go
ch := make(chan int)
go func() {
    ch <- 42 // kanala değer gönder
}()
deger := <-ch // kanaldan değer al
```

## Örnek Kod

Aşağıdaki örnek, birden fazla URL'nin durumunu `sync.WaitGroup` ve goroutine'ler kullanarak eşzamanlı olarak kontrol etmektedir.

```go
package main

import (
	"fmt"
	"net/http"
	"sync"
	"time"
)

// siteKontrol, bir URL'ye istek atar ve durumunu döndürür.
// Sonucu kanal üzerinden gönderir.
func siteKontrol(url string, sonuc chan<- string, wg *sync.WaitGroup) {
	defer wg.Done() // fonksiyon bitince sayacı azalt

	baslangic := time.Now()
	resp, err := http.Get(url)
	if err != nil {
		sonuc <- fmt.Sprintf("%s: HATA (%v)", url, err)
		return
	}
	defer resp.Body.Close()

	sure := time.Since(baslangic)
	sonuc <- fmt.Sprintf("%s: %d (%v)", url, resp.StatusCode, sure)
}

func main() {
	// Kontrol edilecek URL'ler
	urls := []string{
		"https://go.dev",
		"https://www.google.com",
		"https://api.github.com",
	}

	// Sonuçları toplamak için kanal (buffer'li)
	sonucKanali := make(chan string, len(urls))

	// WaitGroup: tüm goroutine'lerin bitmesini beklemek için
	var wg sync.WaitGroup

	// Her URL için bir goroutine başlat
	for _, url := range urls {
		wg.Add(1) // her goroutine için sayacı artır
		go siteKontrol(url, sonucKanali, &wg)
	}

	// Tüm goroutine'lerin bitmesini bekle
	wg.Wait()
	close(sonucKanali) // kanalı kapat (tüm gönderimler bitti)

	// Sonuçları yazdır
	fmt.Println("Eşzamanlı kontrol sonuçları:")
	for mesaj := range sonucKanali {
		fmt.Println("  ", mesaj)
	}
}
```

**Çıktı (örnek):**

```
Eşzamanlı kontrol sonuçları:
   https://go.dev: 200 (234ms)
   https://www.google.com: 200 (412ms)
   https://api.github.com: 200 (189ms)
```

## En İyi Uygulamalar

- **Senkronizasyon için `sync.WaitGroup` kullanın, `time.Sleep` kullanmayın.** `Sleep` keyfi bir süre bekler ve goroutine'in bitip bitmediğini garanti etmez .
- **Döngü değişkenlerini goroutine'lere parametre olarak geçirin.** Go 1.22 öncesinde tüm goroutine'ler aynı değişkeni paylaşırdı; Go 1.22+ ile bu durum `for` döngülerinde düzeltildi, ancak açık parametre geçişi hâlâ daha net ve güvenlidir .
- **Goroutine'lerin temiz bir şekilde sonlanmasını sağlayın.** `main` fonksiyonu bittiğinde tüm goroutine'ler sonlanır; arka plan işlerinin tamamlanmasını beklemek için senkronizasyon kullanın .
- **Paylaşılan veriler için kanalları veya `sync.Mutex` kullanın.** Aynı değişkene eşzamanlı erişim yarış koşullarına yol açar; kanallar veya kilitler ile koruyun .
- **Panik durumlarını yönetin.** Bir goroutine'de oluşan panik tüm programı çökertebilir; `recover` ile yakalayarak diğer goroutine'lerin etkilenmesini önleyin .

> **Sık Yapılan Hatalar ve Uyarılar**
>
> **Döngü değişkenini closure içinde yakalamak**: `for _, url := range urls { go func() { fmt.Println(url) }() }` yazıldığında, tüm goroutine'ler aynı `url` değişkenine bağlanır ve çoğu zaman son değeri yazdırır. Çözüm: değişkeni parametre olarak geçirin (`go func(u string) { ... }(url)`) veya döngü içinde gölgeleyin (`url := url`) . **`time.Sleep` ile senkronizasyon**: Goroutine'in bitmesini beklemek için `time.Sleep` kullanmak güvenilir değildir; süre çok kısa olursa goroutine çalışmaz, çok uzun olursa gereksiz beklemeye yol açar. `sync.WaitGroup` kullanın . **`main` fonksiyonu erken sonlanması**: `main` biterse tüm goroutine'ler anında öldürülür; arka plan işlerinin tamamlandığından emin olun . **`sync.WaitGroup`'u değer olarak geçirmek**: Fonksiyona `wg sync.WaitGroup` (pointer değil) geçirildiğinde, kopya üzerinde çalışılır ve orijinal sayaç güncellenmez; `*sync.WaitGroup` kullanın .

## Kaynakça

- A Tour of Go – Goroutines: https://go.dev/tour/concurrency/1
- Go Paket Dokümantasyonu – `sync`: https://pkg.go.dev/sync
- Go Wiki – Common Mistakes (Goroutines): https://go.dev/wiki/CommonMistakes

## Sonuç

Go'da goroutine'ler, `go` anahtar kelimesiyle başlatılan hafif eşzamanlı iş parçacıklarıdır. OS thread'lerinden çok daha az kaynak tüketir ve Go runtime tarafından yönetilir. Arka plan işlemleri, paralel hesaplama ve eşzamanlı G/Ç işlemleri için idealdir. Ancak çalıştırma sırası garanti edilmez; senkronizasyon için `sync.WaitGroup`, veri paylaşımı için kanallar veya `sync.Mutex` kullanılmalıdır. Döngü değişkenlerini closure'larda doğru yakalamak, `main` fonksiyonunun erken sonlanmasını önlemek ve paylaşılan verileri korumak, güvenilir eşzamanlı Go programlarının temelini oluşturur.
