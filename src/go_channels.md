# GO - Channels

## Channels Nedir?

Go dilinde kanal (channel), goroutine'ler arasında veri alışverişi yapmak için kullanılan **tipli bir boru hattıdır** (typed conduit) . Kanallar, Go'nun eşzamanlılık modelinin temel taşıdır ve "hafızayı paylaşarak iletişim kurmayın; iletişim kurarak hafızayı paylaşın" felsefesini hayata geçirir. Bir goroutine kanala değer gönderir, başka bir goroutine aynı kanaldan bu değeri alır. Bu mekanizma, paylaşılan belleğe doğrudan erişim yerine güvenli bir veri akışı sağlar.

Kanallar **referans türüdür**; yani `make` ile oluşturulur ve fonksiyonlara pointer gibi geçirilebilir. Sıfır değeri `nil`'dir ve `nil` kanal üzerinde gönderme veya alma işlemi **sonsuza kadar bloklanır** .

## Sözdizimi (Syntax)

Kanal oluşturmak için `make` fonksiyonu ve `chan` anahtar kelimesi kullanılır:

```go
// Tamponsuz (unbuffered) kanal
ch := make(chan int)

// Tamponlu (buffered) kanal, kapasite 10
ch := make(chan int, 10)
```

Kanala değer göndermek ve kanaldan değer almak için `<-` operatörü kullanılır:

```go
ch <- 42       // kanala değer gönder
deger := <-ch  // kanaldan değer al
```

Kanallar yönlü olarak da tanımlanabilir. Bu, özellikle fonksiyon imzalarında sorumlulukları netleştirmek için kullanışlıdır :

```go
var sadeceGonder chan<- int  // sadece gönderme
var sadeceAl <-chan int      // sadece alma
var ciftYonlu chan int       // her iki yön
```

## Kullanım Alanları

- **Goroutine'ler arası veri paylaşımı**: Hesaplama sonuçlarını bir goroutine'den diğerine güvenle aktarmak.
- **Senkronizasyon**: Tamponsuz kanallar, gönderen ve alan tarafın aynı anda hazır olmasını gerektirerek doğal bir senkronizasyon noktası oluşturur .
- **İş akışı kontrolü (Semaphore)**: Tamponlu kanallar, eşzamanlı işlem sayısını sınırlamak için semafor olarak kullanılabilir .
- **Olay sinyali**: `chan struct{}` türünde kanallar, veri taşımadan yalnızca "işlem tamamlandı" veya "dur" sinyali göndermek için idealdir .

## Kullanım Türleri

**1. Tamponsuz Kanal (Unbuffered)**

Gönderen goroutine, alıcı hazır olana kadar bloklanır. Bu, verinin alıcı tarafından alındığının **%100 garantisini** verir .

```go
ch := make(chan string)
go func() { ch <- "merhaba" }()
fmt.Println(<-ch) // "merhaba"
```

**2. Tamponlu Kanal (Buffered)**

Tampon dolana kadar gönderen bloklanmaz. Tampon dolduğunda gönderen, alıcı bir değer alana kadar bekler .

```go
ch := make(chan int, 2)
ch <- 1 // bloklanmaz
ch <- 2 // bloklanmaz
// ch <- 3 // tampon dolu, bloklanır
```

**3. Kanalları Kapatma (`close`)**

Gönderen taraf, başka değer göndermeyeceğini bildirmek için kanalı kapatabilir. Alıcı taraf, `v, ok := <-ch` ile kanalın kapanıp kapanmadığını kontrol edebilir. `ok` değeri `false` ise kanal kapatılmıştır .

```go
close(ch)
v, ok := <-ch // ok == false
```

**Kritik kural**: Kanalı **yalnızca gönderen taraf** kapatmalıdır. Kapalı bir kanala gönderme yapmak `panic`'e yol açar .

**4. `range` ile Kanal Üzerinde Döngü**

`for v := range ch` yapısı, kanal kapanana kadar değerleri almaya devam eder. Bu, kanal tüketimi için en temiz yöntemdir .

```go
go func() {
    for i := 0; i < 3; i++ { ch <- i }
    close(ch)
}()
for v := range ch {
    fmt.Println(v)
}
```

**5. `select` ile Çoklu Kanal Yönetimi**

`select`, birden fazla kanal işlemini aynı anda beklemeyi sağlar. Hangi `case` bloklanmıyorsa o çalışır. `default` bloğu, hiçbir kanal hazır değilse çalışır ve bloklamayı önler .

```go
select {
case v := <-ch1:
    fmt.Println("ch1:", v)
case v := <-ch2:
    fmt.Println("ch2:", v)
default:
    fmt.Println("hazır kanal yok")
}
```

## Örnek Kod

Aşağıdaki örnek, bir işçi havuzu (worker pool) senaryosunda tamponlu kanal, `range` ve `close` kullanımını göstermektedir.

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// isci, is kanalindan gelen isleri isler ve sonucu sonuc kanalina yazar.
func isci(id int, isler <-chan int, sonuclar chan<- string, wg *sync.WaitGroup) {
	defer wg.Done() // isci bitince sayaci azalt

	// range, kanal kapanana kadar deger almaya devam eder
	for is := range isler {
		// Isi isleme simülasyonu
		time.Sleep(100 * time.Millisecond)
		sonuclar <- fmt.Sprintf("İşçi %d: iş %d tamamlandı", id, is)
	}
}

func main() {
	// Tamponlu kanallar: isler icin 10 kapasite, sonuclar icin 10 kapasite
	isler := make(chan int, 10)
	sonuclar := make(chan string, 10)

	var wg sync.WaitGroup

	// 3 isci baslat
	for i := 1; i <= 3; i++ {
		wg.Add(1)
		go isci(i, isler, sonuclar, &wg)
	}

	// Isleri kanala gonder
	for i := 1; i <= 6; i++ {
		isler <- i
	}
	close(isler) // isler kanalini kapat: artik is gelmeyecek

	// Isçilerin bitmesini bekle, sonra sonuclar kanalini kapat
	go func() {
		wg.Wait()
		close(sonuclar)
	}()

	// Sonuclari yazdir
	for sonuc := range sonuclar {
		fmt.Println(sonuc)
	}
}
```

**Çıktı (örnek):**

```
İşçi 1: iş 1 tamamlandı
İşçi 2: iş 2 tamamlandı
İşçi 3: iş 3 tamamlandı
İşçi 1: iş 4 tamamlandı
İşçi 2: iş 5 tamamlandı
İşçi 3: iş 6 tamamlandı
```

## En İyi Uygulamalar

- **Kanalı yalnızca gönderen taraf kapatın.** Alıcı tarafın kanalı kapatması, gönderenin kapalı kanala yazmasına ve `panic`'e yol açabilir .
- **Tampon boyutunu dikkatli seçin.** Çoğu durumda 0 (tamponsuz) veya 1 idealdir. 0, koordinasyon için; 1, gönderenin bloklanmadan devam edebilmesi için uygundur. Büyük tamponlar gecikmeyi artırabilir ve geri basınç (backpressure) sinyalini geciktirir .
- **`range` kullanın, `for-select` yerine.** Tek bir kanaldan sürekli okuma yapılıyorsa, `for v := range ch` hem daha kısa hem de kanal kapanışını otomatik algılar .
- **`chan struct{}` kullanın, sinyal için.** Yalnızca "dur" veya "tamamlandı" sinyali gönderilecekse, `struct{}` türünde kanal kullanmak niyeti netleştirir ve bellek tasarrufu sağlar .
- **`select` ile timeout ekleyin.** Ağ işlemlerinde `time.After` ile birlikte `select` kullanarak sonsuz beklemeyi önleyin.

> **Sık Yapılan Hatalar ve Uyarılar**
>
> **`nil` kanal üzerinde işlem yapmak**: `var ch chan int` tanımıyla oluşturulan ve `make` ile başlatılmayan kanal `nil`'dir. `nil` kanaldan okuma veya `nil` kanala yazma **sonsuza kadar bloklanır** ve deadlock'a yol açar . **Kapalı kanala yazmak**: `close(ch)` sonrasında `ch <- v` yazmak `panic: send on closed channel` hatası verir . **Kapalı kanaldan okumaya devam etmek**: Kanal kapandıktan sonra okuma, türün **sıfır değerini** döndürür; `ok` değişkeni `false` olur. `range` döngüsü bu durumu otomatik algılar ve sonlanır . **Aynı kanalı birden fazla goroutine'in tüketmesi**: Hangi goroutine'in hangi değeri alacağı **belirsizdir** (non-deterministic). Başlangıçta tek tüketici kullanmak, mantık hatalarını önler . **`sync.WaitGroup`'u kanal yerine kullanmak**: Kanal veri akışı için, `WaitGroup` ise tamamlanma beklemesi içindir; ikisini karıştırmak gereksiz karmaşıklık yaratır.

## Kaynakça

- A Tour of Go – Channels: https://go.dev/tour/concurrency/2
- A Tour of Go – Range and Close: https://go.dev/tour/concurrency/4
- Effective Go – Channels: https://go.dev/doc/effective_go#channels
- Dave Cheney – Practical Go (Channel Sizes): https://dave.cheney.net/practical-go
- Freie Universität Berlin – Go Channels: https://www.inf.fu-berlin.de/inst/ag-se/teaching/K-ProPra-2026-04/go-channels.html

## Sonuç

Go'da kanallar, goroutine'ler arasında güvenli veri iletişimi sağlayan tipli boru hatlarıdır. Tamponsuz kanallar güçlü senkronizasyon garantisi sunarken, tamponlu kanallar üretici ve tüketici arasında esneklik sağlar. `close` ile kanal kapatma, `range` ile otomatik tüketim ve `select` ile çoklu kanal yönetimi, Go'nun eşzamanlılık araç setinin temelini oluşturur. Kanalı yalnızca gönderen tarafın kapatması, `nil` kanal tuzaklarından kaçınmak ve tampon boyutunu bilinçli seçmek, güvenilir eşzamanlı Go programlarının anahtarıdır.
