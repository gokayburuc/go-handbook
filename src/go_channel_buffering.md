# GO - Channel Buffering

## Channel Buffering Nedir?

Go dilinde kanal tamponlama, bir kanalın **alıcı hazır olmadan belirli sayıda değeri geçici olarak depolayabilmesi** özelliğidir. Varsayılan olarak kanallar **tamponsuzdur** (unbuffered); yani bir gönderme işlemi ancak karşılık gelen bir alma işlemi hazır olduğunda gerçekleşir, aksi halde gönderen goroutine bloklanır . Tamponlu kanallar ise `make(chan T, kapasite)` sözdizimiyle oluşturulur ve tampon dolana kadar gönderen tarafın bloklanmadan devam etmesine olanak tanır .

Tamponlu bir kanal, eşzamanlı bir kuyruk (queue) gibi davranır: gönderilen değerler sıraya eklenir, alınan değerler sıradan çıkarılır. Tampon dolduğunda gönderme işlemi, alıcı bir değer çekene kadar bekler; tampon boşken alma işlemi, gönderen bir değer ekleyene kadar bekler .

## Sözdizimi (Syntax)

Tamponlu kanal oluşturmak için `make` fonksiyonuna ikinci bir argüman olarak kapasite verilir:

```go
// Kapasitesi 5 olan tamponlu kanal
ch := make(chan int, 5)

// Tamponsuz kanal (kapasite 0)
ch := make(chan int)
```

Kapasite, tamponun **alabileceği maksimum eleman sayısını** belirler. Bu değer, `cap(ch)` fonksiyonu ile sorgulanabilir .

## Kullanım Alanları

- **Üretici-tüketici deseni**: Üretici, tüketici henüz hazır değilken bile tampona veri bırakabilir; bu, iki tarafın hızlarının tam eşleşmesini gerektirmez .
- **İş dağıtımı (worker pool)**: Görevler tamponlu bir kanala yazılır ve sabit sayıda işçi goroutine bu kanaldan görev alır. Tampon boyutu genellikle işçi sayısına yakın seçilir .
- **Semafor / eşzamanlılık sınırlama**: Tamponlu bir kanal, aynı anda çalışabilecek işlem sayısını sınırlamak için kullanılabilir. Örneğin `make(chan struct{}, 10)` ile en fazla 10 eşzamanlı işleme izin verilir .
- **Sinyal / bildirim**: Kapasitesi 1 olan `chan struct{}` tipi kanallar, veri taşımadan yalnızca "tamamlandı" veya "hazır" sinyali göndermek için idealdir .

## Kullanım Türleri

**1. Tamponsuz Kanal (Varsayılan)**

Gönderen, alıcı hazır olana kadar bloklanır. Bu, verinin alıcıya **ulaştığının %100 garantisini** verir ve güçlü bir senkronizasyon sağlar .

```go
ch := make(chan int)
go func() { ch <- 42 }()
fmt.Println(<-ch)
```

**2. Tamponlu Kanal (Kapasite > 0)**

Tampon dolana kadar gönderen bloklanmaz. Bu, gönderen ve alıcıyı **ayrıştırır** (decouple) .

```go
ch := make(chan int, 2)
ch <- 1 // bloklanmaz
ch <- 2 // bloklanmaz
// ch <- 3 // tampon dolu, bloklanır
```

**3. Semafor Olarak Kullanım**

Tampon kapasitesi, eşzamanlı işlem sayısını sınırlar :

```go
sem := make(chan struct{}, 3) // en fazla 3 eşzamanlı işlem
for i := 0; i < 10; i++ {
    go func(id int) {
        sem <- struct{}{}        // izin al
        defer func() { <-sem }() // izni bırak
        // işlemi gerçekleştir
    }(i)
}
```

## Örnek Kod

Aşağıdaki örnek, tamponlu kanalın üretici-tüketici deseninde nasıl kullanıldığını ve tampon dolduğunda gönderenin nasıl bloklandığını göstermektedir.

```go
package main

import (
	"fmt"
	"time"
)

// uretici, 5 adet deger uretir ve tamponlu kanala yazar.
// Tampon doldugunda gonderim bloklanir.
func uretici(ch chan<- int) {
	for i := 1; i <= 5; i++ {
		fmt.Printf("Üretici: %d gönderiliyor\n", i)
		ch <- i // tampon doluysa burada bekler
		fmt.Printf("Üretici: %d tampona yazıldı\n", i)
	}
	close(ch)
}

func main() {
	// Kapasitesi 2 olan tamponlu kanal
	ch := make(chan int, 2)

	go uretici(ch)

	// Tuketici yavas: her okumadan once bekler
	for deger := range ch {
		fmt.Printf("Tüketici: %d alındı\n", deger)
		time.Sleep(500 * time.Millisecond) // isleme suresi
	}
}
```

**Çıktı (örnek):**

```
Üretici: 1 gönderiliyor
Üretici: 1 tampona yazıldı
Üretici: 2 gönderiliyor
Üretici: 2 tampona yazıldı
Üretici: 3 gönderiliyor
Tüketici: 1 alındı
Üretici: 3 tampona yazıldı
Üretici: 4 gönderiliyor
Tüketici: 2 alındı
Üretici: 4 tampona yazıldı
Üretici: 5 gönderiliyor
Tüketici: 3 alındı
Üretici: 5 tampona yazıldı
Tüketici: 4 alındı
Tüketici: 5 alındı
```

Görüldüğü gibi, tampon (kapasite 2) dolduğunda üretici 3. değeri göndermek için beklemiş, tüketici bir değer çektikten sonra devam edebilmiştir.

## En İyi Uygulamalar

- **Tampon boyutunu amaca göre seçin.** Sinyal için 1, iş dağıtımı için işçi sayısı, toplu işleme için parti boyutu uygun değerlerdir. `make(chan int, 40)` gibi gerekçesiz "sihirli sayılar" kullanmaktan kaçının .
- **Varsayılan olarak küçük tampon kullanın.** Emin değilseniz, 1 veya tamponsuz kanalla başlayın. Büyük tamponlar yalnızca performans sorunlarını gizler, çözmez .
- **Backpressure sinyalini görünür tutun.** Tampon, üretici ve tüketici arasındaki hız farkını **geçici olarak** yumuşatmak içindir; sürekli bir hız uyumsuzluğunu gizlemek için değil. Tampon sürekli doluysa, çözüm tamponu büyütmek değil, tüketici sayısını veya işlem hızını artırmaktır .
- **Tampon kullanımını izleyin.** `len(ch)/cap(ch)` oranını metrik olarak dışa verin; bu oran sürekli yüksekse tüketici tarafı yetersiz demektir .

> **Sık Yapılan Hatalar ve Uyarılar**
>
> - **Aşırı büyük tampon backpressure'ı gizler.** 10.000 kapasiteli bir kanal, tüketicinin yetişemediği gerçeğini örter; sorun yalnızca geciktirilir ve bellek tüketimi artar .
> - **Tampon boyutunu gerekçesiz seçmek** (`make(chan int, 40)`) bakımı zor ve niyeti belirsiz kod üretir; boyut, işçi sayısı veya ölçülen bir değere bağlanmalıdır .
> - **Tamponsuz kanalı yanlışlıkla kullanmak**, gönderenin alıcı hazır olana kadar beklemesine ve beklenmedik bloklanmalara yol açabilir .
> - **Büyük tamponun performansı her zaman artırdığını sanmak** yanlıştır; tampon yalnızca düzensiz/burst yüklerde anlamlıdır, sürekli hız uyumsuzluğunda çözüm değildir .

## Kaynakça

- Effective Go – Channels: https://go.dev/doc/effective_go#channels
- Go Turu – Buffered Channels: https://go.dev/tour/concurrency/3
- Microsoft Learn – Buffered Channels: https://learn.microsoft.com/en-us/training/modules/go-concurrency/3-buffered-channels

## Sonuç

Go'da kanal tamponlama, gönderen ve alıcı goroutine'leri arasındaki doğrudan senkronizasyon bağımlılığını gevşetir. Tamponsuz kanallar güçlü senkronizasyon ve teslim garantisi sunarken, tamponlu kanallar üretici ile tüketici arasında esneklik sağlar. Tampon boyutu; sinyal (1), iş dağıtımı (işçi sayısı) veya toplu işleme (parti boyutu) gibi somut kullanım amacına göre belirlenmelidir. Büyük tamponlar backpressure sinyalini gizleyerek sorunları derinleştirebilir; bu nedenle tampon, geçici dalgalanmaları yumuşatmak için kullanılmalı, sürekli hız uyumsuzluklarının çözümü olarak görülmemelidir.
