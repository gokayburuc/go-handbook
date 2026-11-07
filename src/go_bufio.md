# GO - Bufio Library

# GO Bufio Library

Go'da dosya, terminal veya ağ bağlantılarından veri okurken her karakter veya byte için doğrudan işletim sistemi çağrısı yapmak verimsizdir.

`bufio` (**buffered I/O**) paketi, veri okuma ve yazma işlemlerini bir tampon (buffer) üzerinden gerçekleştirerek performansı artırır.

Paket içerisindeki en sık kullanılan yapılar:

- `bufio.Reader` → Veriyi tamponlu okur.
- `bufio.Writer` → Veriyi tamponlu yazar.
- `bufio.Scanner` → Satır satır veya kelime kelime veri okur.

## Kullanım Alanları

### Kullanıcıdan Veri Almak ( os.stdin)

Kullanıcıdan `os.Stdin` gibi _standart input_ kütüphanesinden veri almak için kullanılır.

```go
reader := bufio.NewReader(os.Stdin)
```

### Büyük Dosyaları Satır Satır Okumak İçin

Özellikle büyük dosyaları tamamiyle hafızaya alıp performansı düşürmemek adına satır satır okuma işlemi yapılacağı zaman kullanılır.

```go
scanner := bufio.NewScanner(file)
```

### TCP Sunucularından Gelen Verileri Okumak için

[[TCP-Transmission-Control-Protocol]] yapısında [[web-socket]] üzerinden gelen verileri okumak için kullanılır.

```go
reader := bufio.NewReader(conn)
```

### Log Dosyalarını Satır Satır İşleme

Büyük log dosyalarını satır satır analiz etmek için kullanılır.

- [ ] Log dosyaları Bufio ile nasıl okunur?

## Kullanım Örneği

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	reader := bufio.NewReader(os.Stdin)
	fmt.Print("İsminiz: ")
	name, _ := reader.ReadString('\n')
	fmt.Println("Merhaba", name)
}
```

Yukarıdaki kod terminal üzerinden `İsminiz:` olarak giriş ister ve bunu `name` isimli değişkene atar. Kod çıktısı aşağıdaki gibi olabilir:

```bash
İsminiz: Gokay
Merhaba Gokay
```

## Bilgilendirme

Backend geliştirmede `bufio.Scanner` ve `bufio.Reader` çok sık karşınıza çıkar:

- Log işleyiciler
- CSV okuyucular
- TCP sunucuları
- CLI uygulamaları
- ETL süreçleri
- Veri işleme servisleri

Özellikle dosya ve ağ bağlantılarıyla çalışırken `bufio` kullanmak, doğrudan `os.File` veya `net.Conn` üzerinden byte byte okumaya göre çok daha verimlidir.
