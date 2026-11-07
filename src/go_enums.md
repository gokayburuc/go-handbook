# GO - Enums

## Enums (Numaralandırma) Nedir?

Go dilinde **yerleşik bir `enum` anahtar kelimesi yoktur**. Bunun yerine, numaralandırma benzeri yapılar `const` bloğu, `iota` sabit üreteci ve isteğe bağlı olarak tanımlanmış bir tür (named type) bir araya getirilerek oluşturulur . Bu yaklaşım, bir grup ilişkili sabit değeri tek bir tür altında toplayarak "sihirli sayıların" kod içinde dağınık şekilde kullanılmasını önler ve derleyici düzeyinde bir miktar tür güvenliği sağlar.

Ancak bu yapı, diğer dillerdeki `enum` kadar katı değildir. Tanımlı bir tür (`type Role int`) kullanılsa bile, `Role(42)` gibi tür dönüşümleriyle tanımsız değerler atanabilir; derleyici bunu engellemez . Bu nedenle Go'da enum, bir **sözleşme** olarak değil, bir **gelenek** olarak düşünülmelidir.

## Sözdizimi (Syntax)

Go'da enum oluşturmanın deyimsel (idiomatic) yolu şu üç adımdan oluşur:

1.  **Temel tür tanımlama**: `type TurAdi int` (veya `string`).
2.  **`iota` ile sabit bloğu**: Her sabit, `iota` sayesinde otomatik artan bir değer alır.
3.  **`String()` metodu**: Değerlerin okunabilir biçimde yazdırılması için `fmt.Stringer` arayüzü uygulanır .

```go
type Durum int

const (
    Beklemede Durum = iota // 0
    Hazirlaniyor           // 1
    Kargoda                // 2
    TeslimEdildi           // 3
)
```

`iota`, her `const` bloğunun başında sıfırlanır ve her satırda bir artar . İlk değeri sıfırdan farklı yapmak için `iota + 1` veya `100 + iota` gibi ifadeler kullanılabilir .

## Kullanım Alanları

- **Durum makineleri**: Sipariş durumu, bağlantı durumu, sunucu durumu gibi sonlu durum kümelerini modellemek .
- **Log seviyeleri**: `Error`, `Warn`, `Info`, `Debug` gibi sabit kategoriler.
- **Rol ve izin yönetimi**: `Guest`, `Member`, `Admin` gibi kullanıcı rolleri.
- **Bit bayrakları (Bit Flags)**: `1 << iota` ile tek bir baytta birden fazla izni saklamak .

## Kullanım Türleri

**1. Temel `iota` Enum**

En yalın hali. Değerler sıfırdan başlar.

```go
type HaftaGunu int

const (
    Pazartesi HaftaGunu = iota // 0
    Sali                       // 1
    Carsamba                   // 2
)
```

**2. Sıfır Değerini "Bilinmeyen" Olarak Ayırma**

Go'da bir değişkenin sıfır değeri, tanımsız bir durumu temsil etmelidir. Bu nedenle ilk sabiti genellikle `Unknown` veya `Unspecified` yapmak iyi bir uygulamadır .

```go
type LogSeviyesi int

const (
    LogUnknown LogSeviyesi = iota // 0: atanmamış / bilinmeyen
    LogError                      // 1
    LogWarn                       // 2
    LogInfo                       // 3
)
```

**3. `String()` Metodu ile Okunabilirlik**

`fmt.Println` çağrıldığında sayı yerine anlamlı metin yazdırmak için `String()` metodu eklenir .

```go
func (l LogSeviyesi) String() string {
    switch l {
    case LogError:
        return "ERROR"
    case LogWarn:
        return "WARN"
    case LogInfo:
        return "INFO"
    default:
        return "UNKNOWN"
    }
}
```

**4. Bit Bayrakları**

Tek bir tam sayı içinde birden fazla bağımsız özelliği saklamak için `1 << iota` kullanılır .

```go
type Izin int

const (
    IzinOku Izin = 1 << iota // 1 (binary: 001)
    IzinYaz                  // 2 (binary: 010)
    IzinSil                  // 4 (binary: 100)
)

// Birden fazla izni birleştirme
izinler := IzinOku | IzinYaz // 3 (binary: 011)
```

## Örnek Kod

Aşağıdaki örnek, bir e-ticaret siparişinin durumunu `iota` tabanlı bir enum ile modellemekte ve `String()` metodu ile okunabilir çıktı üretmektedir.

```go
package main

import (
	"fmt"
)

// SiparisDurumu, siparişin yaşam döngüsündeki durumları temsil eder.
type SiparisDurumu int

const (
	// DurumBilinmiyor, sıfır değerini temsil eder (atanmamış).
	DurumBilinmiyor SiparisDurumu = iota
	// DurumBeklemede, siparişin alındığını ancak işleme başlanmadığını belirtir.
	DurumBeklemede
	// DurumHazirlaniyor, siparişin paketlendiğini belirtir.
	DurumHazirlaniyor
	// DurumKargoda, siparişin kargoya verildiğini belirtir.
	DurumKargoda
	// DurumTeslimEdildi, siparişin müşteriye ulaştığını belirtir.
	DurumTeslimEdildi
)

// String, SiparisDurumu değerini okunabilir metne dönüştürür.
// fmt.Stringer arayüzünü uygular; fmt.Println otomatik olarak bunu çağırır.
func (d SiparisDurumu) String() string {
	switch d {
	case DurumBeklemede:
		return "Beklemede"
	case DurumHazirlaniyor:
		return "Hazırlanıyor"
	case DurumKargoda:
		return "Kargoda"
	case DurumTeslimEdildi:
		return "Teslim Edildi"
	default:
		return fmt.Sprintf("Bilinmeyen(%d)", int(d))
	}
}

// DurumAciklamasi, duruma göre müşteriye gösterilecek mesajı döndürür.
func DurumAciklamasi(d SiparisDurumu) string {
	switch d {
	case DurumBeklemede:
		return "Siparişiniz alındı, onay bekleniyor."
	case DurumHazirlaniyor:
		return "Siparişiniz hazırlanıyor."
	case DurumKargoda:
		return "Siparişiniz yolda."
	case DurumTeslimEdildi:
		return "Siparişiniz teslim edildi."
	default:
		return "Geçersiz sipariş durumu."
	}
}

func main() {
	// Enum değerleri oluşturma
	siparis1 := DurumBeklemede
	siparis2 := DurumKargoda

	// String() metodu sayesinde okunabilir çıktı
	fmt.Printf("Sipariş 1 durumu: %v\n", siparis1)
	fmt.Printf("Sipariş 2 durumu: %v\n", siparis2)

	// Durum açıklamaları
	fmt.Println(DurumAciklamasi(siparis1))
	fmt.Println(DurumAciklamasi(siparis2))

	// Sayısal değere erişim
	fmt.Printf("Kargoda sayısal değeri: %d\n", int(DurumKargoda))

	// Tanımsız bir değer (derleyici engellemez, String() yakalar)
	gecersiz := SiparisDurumu(99)
	fmt.Printf("Geçersiz değer: %v\n", gecersiz)
}
```

**Çıktı:**

```
Sipariş 1 durumu: Beklemede
Sipariş 2 durumu: Kargoda
Siparişiniz alındı, onay bekleniyor.
Siparişiniz yolda.
Kargoda sayısal değeri: 3
Geçersiz değer: Bilinmeyen(99)
```

## En İyi Uygulamalar

- **`iota` ile başlayan ilk değeri `Unknown` veya `Unspecified` yapın.** Go'da bir değişkenin sıfır değeri, "atanmamış" anlamına gelmelidir; aksi halde `var d Durum` ifadesi sessizce ilk geçerli durumu (örneğin "Beklemede") atar ve hatayı gizler .
- **Kalıcı veriler için açık değerler kullanın.** Veritabanına, log'a veya ağ üzerinden başka bir sisteme gönderilecek enum değerlerinde `iota` yerine açık sayılar (`DurumA = 1`, `DurumB = 2` gibi) tercih edin. Araya yeni bir değer eklemek, `iota` tabanlı tüm sonraki değerleri kaydırır ve kalıcı verileri bozar .
- **`String()` metodunu mutlaka ekleyin.** `fmt.Stringer` arayüzü, log ve hata mesajlarında okunabilirliği artırır. Çok sayıda değer varsa `stringer` aracı (`go install golang.org/x/tools/cmd/stringer@latest`) ile bu metodu otomatik üretebilirsiniz .
- **`iota`'yı yalnızca bellek içi kullanım için tercih edin.** API yanıtlarında veya veritabanı şemalarında `iota` değerlerini doğrudan kullanmayın; okunabilir string değerler veya açık sayılar daha güvenlidir .

> **Sık Yapılan Hatalar ve Uyarılar**
>
> **Tür güvenliği yanılsaması**: `type Rol int` tanımlamak, `Rol(42)` yazılmasını engellemez. Derleyici yalnızca `int` ile `Rol` arasındaki örtük dönüşümü engeller; açık dönüşümle her tam sayı bir enum değeri gibi kullanılabilir . **Sıfır değeri tuzağı**: İlk sabit `Guest = iota` (0) ise, `var r Rol` tanımı otomatik olarak `Guest` atar; bu, "rol atanmadı" durumunu gizler. Çözüm, ilk değeri `Unknown` yapmaktır . **`iota` kaydırma riski**: Kalıcı verilerde `iota` kullanılıyorsa, listeye araya yeni bir sabit eklemek sonraki tüm sayısal değerleri değiştirir ve eski kayıtlarla uyumsuzluk yaratır . **String dönüşümünün eksikliği**: `String()` metodu olmadan `fmt.Println` çıktısı anlamsız sayılar üretir; log ve hata ayıklamada bu durum ciddi karışıklığa yol açar.

## Kaynakça

- Go Dil Spesifikasyonu – Iota: https://go.dev/ref/spec#Iota
- Go Blog – Generating code (stringer): https://go.dev/blog/generate
- `stringer` aracı: https://pkg.go.dev/golang.org/x/tools/cmd/stringer

## Sonuç

Go'da enum, dilin yerleşik bir özelliği değil, `const` + `iota` + isteğe bağlı `String()` metodu ile oluşturulan deyimsel bir kalıptır. Bu yapı, sihirli sayıları ortadan kaldırır ve kodun okunabilirliğini artırır; ancak diğer dillerdeki gibi katı bir tür güvenliği sunmaz. İlk değeri `Unknown` yapmak, kalıcı verilerde `iota` yerine açık değerler kullanmak ve `String()` metodunu eklemek, bu kalıbı güvenli ve sürdürülebilir hale getirir.
