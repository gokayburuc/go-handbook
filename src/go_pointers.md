# GO - Pointers

## Pointers (İşaretçiler) Nedir?

Go dilinde pointer, bir değişkenin değerini değil, o değerin **bellek adresini** tutan bir değişkendir . Pointer'lar, bir fonksiyonun kendi kapsamı dışındaki verilere erişmesini ve bu verileri değiştirmesini sağlar . Go'da pointer türü `*T` şeklinde gösterilir ve `T` türünden bir değere işaret eder; sıfır değeri `nil`'dir .

Go pointer'ları C/C++'tan önemli ölçüde farklıdır: **pointer aritmetiği yoktur**, tür güvenlidir ve çöp toplayıcı (garbage collector) tarafından yönetilir . Bu tasarım, bellek güvenliğini artırırken buffer overflow gibi yaygın hataları ortadan kaldırır .

## Sözdizimi (Syntax)

Pointer tanımlamak ve kullanmak için iki temel operatör vardır: `&` (adres alma) ve `*` (dereference) .

```go
// Pointer tanımlama
var p *int // int türüne işaret eden pointer; sıfır değeri nil

// Adres alma operatörü &
i := 42
p = &i // p artık i'nin bellek adresini tutar

// Dereference operatörü *
fmt.Println(*p) // p'nin işaret ettiği değeri okur: 42
*p = 21         // p üzerinden i'nin değerini değiştirir
```

Struct pointer'larında alan erişimi için açık dereference gerekmez; `p.X` yazmak `(*p).X` ile aynı anlama gelir .

## Kullanım Alanları

Pointer'lar özellikle şu durumlarda tercih edilir :

- **Fonksiyonun çağıranın değerini değiştirmesi gerektiğinde**: Go'da argümanlar değer olarak kopyalanır; orijinali değiştirmek için pointer gerekir.
- **Büyük struct'larda kopyalama maliyetini düşürmek için**: Büyük veri yapılarını pointer ile geçirmek yalnızca adres kopyalar.
- **"Yok" ile "sıfır değeri" ayrımı gerektiğinde**: `*string` gibi nil olabilen pointer'lar, isteğe bağlı alanlar için kullanılır.
- **Mutex, pool veya bağlantı içeren struct'larda**: Bu türleri kopyalamak hataya yol açar; pointer zorunludur.

## Kullanım Türleri

**1. Temel Pointer Kullanımı**

```go
x := 10
p := &x        // p, x'in adresini tutar
*p = 20        // x'in değeri 20 olur
fmt.Println(x) // 20
```

**2. Fonksiyon Parametresi Olarak Pointer**

Fonksiyonun çağıranın değerini değiştirmesini sağlar .

```go
func artir(n *int) {
    *n++
}

sayi := 5
artir(&sayi)
fmt.Println(sayi) // 6
```

**3. Struct Pointer'ı**

Struct alanlarına pointer üzerinden erişimde otomatik dereference yapılır .

```go
type Kullanici struct {
    Ad string
}

k := &Kullanici{Ad: "Ayşe"}
fmt.Println(k.Ad) // Ayşe (açık dereference gerekmez)
```

**4. Method Receiver Olarak Pointer**

Method'un receiver'ı değiştirmesi gerekiyorsa pointer receiver kullanılmalıdır .

```go
type Sayac struct {
    deger int
}

func (s *Sayac) Artir() {
    s.deger++ // orijinal struct değişir
}
```

**5. `new` Fonksiyonu**

`new(T)`, `T` türünde sıfır değerli bir bellek ayırır ve `*T` döndürür. Go'da nadiren kullanılır; struct'lar için `&T{}` tercih edilir .

```go
p := new(int) // *int, sıfır değerli int'ye işaret eder
```

## Örnek Kod

Aşağıdaki örnek, bir banka hesabı üzerinde pointer'lar aracılığıyla para yatırma ve çekme işlemlerini göstermektedir.

```go
package main

import (
	"errors"
	"fmt"
)

// Hesap, bir banka hesabını temsil eder.
type Hesap struct {
	Sahip  string
	Bakiye float64
}

// ParaYatir, hesaba para ekler. Pointer receiver kullanır.
func (h *Hesap) ParaYatir(miktar float64) {
	h.Bakiye += miktar // orijinal hesap değişir
}

// ParaCek, hesaptan para çeker. Yetersiz bakiye kontrolü yapar.
func (h *Hesap) ParaCek(miktar float64) error {
	if miktar > h.Bakiye {
		return errors.New("yetersiz bakiye")
	}
	h.Bakiye -= miktar
	return nil
}

// BakiyeGuncelle, nil kontrolü ile güvenli pointer kullanımını gösterir.
func BakiyeGuncelle(h *Hesap, yeniBakiye float64) error {
	if h == nil {
		return errors.New("hesap nil olamaz")
	}
	h.Bakiye = yeniBakiye
	return nil
}

func main() {
	// Hesap oluşturma: pointer döndürür
	hesap := &Hesap{Sahip: "Ayşe", Bakiye: 1000.0}

	// Pointer receiver ile metod çağrısı
	hesap.ParaYatir(500.0)
	fmt.Printf("Para yatırma sonrası bakiye: %.2f TL\n", hesap.Bakiye)

	// Hata döndüren metod
	if err := hesap.ParaCek(2000.0); err != nil {
		fmt.Printf("Çekme hatası: %v\n", err)
	}

	// Başarılı çekme
	if err := hesap.ParaCek(300.0); err == nil {
		fmt.Printf("Çekme sonrası bakiye: %.2f TL\n", hesap.Bakiye)
	}

	// nil pointer kontrolü
	var bosHesap *Hesap
	if err := BakiyeGuncelle(bosHesap, 100.0); err != nil {
		fmt.Printf("Güncelleme hatası: %v\n", err)
	}

	// Pointer'ların adres karşılaştırması
	kopya := hesap
	kopya.ParaYatir(100.0)
	fmt.Printf("Aynı hesap mı: %t, bakiye: %.2f TL\n", hesap == kopya, hesap.Bakiye)
}
```

**Çıktı:**

```
Para yatırma sonrası bakiye: 1500.00 TL
Çekme hatası: yetersiz bakiye
Çekme sonrası bakiye: 1200.00 TL
Güncelleme hatası: hesap nil olamaz
Aynı hesap mı: true, bakiye: 1300.00 TL
```

## En İyi Uygulamalar

- **Küçük türler için değer, büyük struct'lar için pointer kullanın.** Kopyalama maliyeti düşük olan küçük türlerde pointer gereksiz karmaşıklık yaratır .
- **Değeri değiştirmek gerekiyorsa pointer receiver kullanın.** Method receiver'ın değişmesi gerekiyorsa pointer zorunludur; aksi halde değişiklik kopyada kalır .
- **Tutarlılık sağlayın.** Bir türün bazı metodları pointer receiver kullanıyorsa, diğerleri de kullanmalıdır; aksi halde method set'i karışır .
- **Nil kontrolünü ihmal etmeyin.** Pointer parametrelerini kullanmadan önce `nil` kontrolü yapın; özellikle dış kaynaklardan gelen verilerde bu kritiktir .
- **Slice, map ve channel'lar zaten referans türleridir.** Bu türler için genellikle ayrıca pointer kullanmaya gerek yoktur .

> **Sık Yapılan Hatalar ve Uyarılar**
>
> **Nil pointer dereference en yaygın hatadır.** `var p *int` ile tanımlanan ve başlatılmayan bir pointer'ı dereference etmek (`*p`) çalışma zamanında `panic: runtime error: invalid memory address or nil pointer dereference` hatasına yol açar . **Döngü değişkeninin adresini almak** da dikkat gerektirir: `for _, v := range slice { ptrs = append(ptrs, &v) }` yazıldığında, `&v` dilimin elemanının değil, döngü değişkeninin adresini verir; Go 1.22 sonrası bu davranış iyileştirilmiş olsa da, dilim elemanlarının adresini almak için `&slice[i]` kullanmak daha güvenlidir . **Map elemanlarının adresi alınamaz**: `&m[k]` derleme hatası verir; map elemanları yeniden hash sırasında taşınabilir . **`append` sonrası eski pointer'lar geçersiz olabilir**: Dilim büyüdüğünde yeni bir diziye taşınırsa, eski elemanlara işaret eden pointer'lar artık güncel veriyi göstermez .

## Kaynakça

- A Tour of Go – Pointers: https://go.dev/tour/moretypes/1
- Go FAQ – Pointer vs Value Receivers: https://go.dev/doc/faq#pointer_vs_value_receivers
- Go Packages – Pointers (örnek dokümantasyon): https://pkg.go.dev/github.com/soypat/decaffeinator@v0.0.0-20250429203848-f8a3740ff32f/tagalong/017-pointers

## Sonuç

Go'da pointer'lar, bir değerin bellek adresini tutarak fonksiyonlar arasında veri paylaşımını ve orijinal değerin değiştirilmesini sağlar. `&` operatörü adres alır, `*` operatörü dereference yapar. Go pointer'ları C'den farklı olarak pointer aritmetiği içermez ve çöp toplayıcı tarafından yönetilir; bu da bellek güvenliği sağlar. Pointer'lar özellikle büyük struct'larda kopyalama maliyetini düşürmek, fonksiyonların çağıranın değerini değiştirmesini sağlamak ve nil ile sıfır değeri ayrımı gerektiğinde kullanılır. Nil pointer kontrolü, döngü değişkeni adreslemesi ve `append` sonrası pointer geçerliliği gibi konulara dikkat edilmesi gerekir.
