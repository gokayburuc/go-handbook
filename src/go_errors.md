# GO - Errors

## Error Nedir?

Go dilinde hata (error), bir işlemin başarısız olduğunu bildiren ve **açık bir dönüş değeri** olarak iletilen yerleşik bir arayüzdür. Go'da istisna (exception) mekanizması yoktur; `try-catch` blokları yerine fonksiyonlar, son dönüş değeri olarak `error` türünde bir değer döndürür . Bu yaklaşım, hangi fonksiyonların hata döndürebileceğini açıkça gösterir ve hata kontrolünü dilin diğer yapılarıyla (özellikle `if`) tutarlı hale getirir .

`error` türü bir arayüzdür ve tek bir metot içerir:

```go
type error interface {
    Error() string
}
```

Bu arayüzü uygulayan her tür bir hata değeri olarak kullanılabilir . Go'da hatalar "değer" olarak kabul edilir; bu nedenle programlanabilir, sarılabilir ve zincir halinde incelenebilirler .

## Sözdizimi (Syntax)

Hata döndüren bir fonksiyonun imzası şu kalıbı izler:

```go
func FonksiyonAdi(parametreler) (sonucTuru, error) {
    // ...
    if hataDurumu {
        return sifirDeger, errors.New("hata mesajı")
    }
    return gecerliDeger, nil
}
```

Hata yoksa `nil` döndürülür. Çağıran taraf `if err != nil` ile hatayı kontrol eder .

Hata oluşturmanın iki temel yolu vardır:

```go
// 1. errors.New: sabit mesajlı basit hata
err := errors.New("sıfıra bölme hatası")

// 2. fmt.Errorf: biçimlendirilmiş hata mesajı
err := fmt.Errorf("kullanıcı %d bulunamadı", id)
```

Go 1.13'ten itibaren `fmt.Errorf` içinde `%w` fiili kullanılarak hatalar **sarılabilir** (wrap); bu sayede alttaki hata korunur ve `errors.Is` / `errors.As` ile zincir üzerinden incelenebilir .

## Kullanım Alanları

- **Fonksiyonların başarısızlık durumunu bildirmesi**: Dosya açma, ağ isteği, veritabanı sorgusu gibi işlemlerde.
- **Sentinel hatalar (Sentinel Errors)**: `ErrNotFound` gibi önceden tanımlanmış, `errors.Is` ile kontrol edilebilen belirli hata durumları .
- **Özel hata türleri (Custom Errors)**: Hata hakkında yapılandırılmış veri (kod, alan adı, offset) taşımak gerektiğinde .
- **Hata sarmalama (Error Wrapping)**: Alt katmandan gelen hataya bağlam eklerken orijinal hatayı korumak .

## Kullanım Türleri

**1. Temel Hata Döndürme ve Kontrol**

```go
func Bol(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("sıfıra bölme yapılamaz")
    }
    return a / b, nil
}

// Çağıran taraf
sonuc, err := Bol(10, 0)
if err != nil {
    fmt.Println("Hata:", err)
}
```

**2. Sentinel Error**

Paket düzeyinde tanımlanan, belirli bir durumu temsil eden hata değeridir. İsimlendirme geleneği `Err` öneki iledir .

```go
var ErrBulunamadi = errors.New("kayıt bulunamadı")

func KullaniciGetir(id int) (*Kullanici, error) {
    if k == nil {
        return nil, ErrBulunamadi
    }
    return k, nil
}

// Kontrol
if errors.Is(err, ErrBulunamadi) {
    // özel işlem
}
```

**3. Hata Sarmalama (`%w`)**

Alt katmandan gelen hataya bağlam eklerken `%w` kullanılır; `%v` veya `%s` kullanmak hata zincirini koparır .

```go
func DosyaOku(yol string) ([]byte, error) {
    veri, err := os.ReadFile(yol)
    if err != nil {
        return nil, fmt.Errorf("dosya okunamadı %s: %w", yol, err)
    }
    return veri, nil
}
```

**4. Özel Hata Türü (`errors.As`)**

Yapılandırılmış bilgi taşıyan hatalar için struct tanımlanır .

```go
type DogrulamaHatasi struct {
    Alan   string
    Mesaj  string
}

func (e *DogrulamaHatasi) Error() string {
    return fmt.Sprintf("%s: %s", e.Alan, e.Mesaj)
}

// Kontrol
var dh *DogrulamaHatasi
if errors.As(err, &dh) {
    fmt.Println("Geçersiz alan:", dh.Alan)
}
```

**5. `errors.Is` ve `errors.As`**

`errors.Is`, bir hatanın (veya zincirindeki herhangi bir hatanın) belirli bir değerle eşleşip eşleşmediğini kontrol eder. `errors.As`, hata zincirinde belirli bir türe ulaşmaya çalışır .

## Örnek Kod

Aşağıdaki örnek, bir dosya işleme senaryosunda temel hata kontrolü, sentinel error, sarmalama ve özel hata türünü bir arada göstermektedir.

```go
package main

import (
	"errors"
	"fmt"
	"os"
	"strings"
)

// Sentinel hatalar: paket duzeyinde tanimlanir, Err oneki ile adlandirilir.
var (
	ErrGecersizAd = errors.New("geçersiz isim")
	ErrDosyaYok   = errors.New("dosya bulunamadı")
)

// DogrulamaHatasi, alan bazli dogrulama hatalarini tasir.
type DogrulamaHatasi struct {
	Alan  string
	Mesaj string
}

func (e *DogrulamaHatasi) Error() string {
	return fmt.Sprintf("%s: %s", e.Alan, e.Mesaj)
}

// KullaniciDogrula, isim bos ise ozel hata dondurur.
func KullaniciDogrula(ad string) error {
	if strings.TrimSpace(ad) == "" {
		return &DogrulamaHatasi{Alan: "ad", Mesaj: "boş olamaz"}
	}
	if len(ad) < 2 {
		return ErrGecersizAd // sentinel error
	}
	return nil
}

// DosyaOku, dosyayi okur ve hatayi baglamla sarar.
func DosyaOku(yol string) (string, error) {
	veri, err := os.ReadFile(yol)
	if err != nil {
		// %w ile sarmalama: alttaki hata korunur
		return "", fmt.Errorf("dosya okunamadı %q: %w", yol, err)
	}
	return string(veri), nil
}

func main() {
	// 1. Temel hata kontrolu
	if err := KullaniciDogrula(""); err != nil {
		fmt.Println("Dogrulama hatasi:", err)

		// errors.As: ozel hata turune erisim
		var dh *DogrulamaHatasi
		if errors.As(err, &dh) {
			fmt.Printf("  -> Alan: %s\n", dh.Alan)
		}
	}

	// 2. Sentinel error kontrolu (errors.Is)
	if err := KullaniciDogrula("A"); err != nil {
		if errors.Is(err, ErrGecersizAd) {
			fmt.Println("Sentinel hata yakalandi:", err)
		}
	}

	// 3. Hata sarmalama ve zincir kontrolu
	_, err := DosyaOku("/olmayan/dosya.txt")
	if err != nil {
		fmt.Println("Dosya hatasi:", err)
		// errors.Is ile zincirdeki os.ErrNotExist kontrolu
		if errors.Is(err, os.ErrNotExist) {
			fmt.Println("  -> Kok neden: dosya mevcut degil")
		}
	}
}
```

**Çıktı:**

```
Dogrulama hatasi: ad: boş olamaz
  -> Alan: ad
Sentinel hata yakalandi: geçersiz isim
Dosya hatasi: dosya okunamadı "/olmayan/dosya.txt": open /olmayan/dosya.txt: no such file or directory
  -> Kok neden: dosya mevcut degil
```

## En İyi Uygulamalar

- **`error` her zaman son dönüş değeri olsun.** `(sonuc, error)` kalıbı Go'nun yerleşik geleneğidir .
- **Hata mesajları küçük harfle başlasın ve nokta ile bitmesin.** `"dosya okunamadı"` doğru, `"Dosya okunamadı."` yanlıştır.
- **Bağlam eklerken `%w` kullanın.** `%v` veya `%s` hata zincirini koparır ve `errors.Is` / `errors.As` çalışmaz .
- **Sentinel hataları `Err` öneki ile adlandırın.** `ErrNotFound`, `ErrInvalidInput` gibi .
- **`errors.Is` ve `errors.As` kullanın, `==` veya tür assertion yerine.** Sarmalanmış hatalarda doğru sonuç verir .

> **Sık Yapılan Hatalar ve Uyarılar**
>
> **`%v` veya `%s` ile sarmalama**, hata zincirini koparır; `errors.Is(err, os.ErrNotExist)` gibi kontroller başarısız olur. Bağlam eklerken `%w` kullanın . **Hata kontrolünü atlayıp `_` ile yutmak**, oluşabilecek sorunları sessizce gizler. **Sentinel hataları `==` ile karşılaştırmak**, sarmalanmış hatalarda yanlış sonuç verir; `errors.Is` kullanın . **`panic`'i normal hata yönetimi için kullanmak**, Go'nun felsefesine aykırıdır; `panic` yalnızca programın devam edemeyeceği durumlar içindir.

## Kaynakça

- Go Blog – Error Handling and Go: https://go.dev/blog/error-handling-and-go
- Go Blog – Working with Errors in Go 1.13: https://go.dev/blog/go1.13-errors
- Go by Example – Errors: https://gobyexample.com/errors
- Go Blog – Errors are values: https://go.dev/blog/errors-are-values

## Sonuç

Go'da hatalar, istisna mekanizması yerine açık dönüş değerleri olarak iletilir. `error` arayüzü, `errors.New`, `fmt.Errorf`, sentinel hatalar ve özel hata türleri aracılığıyla hata bilgisi yapılandırılabilir. Go 1.13 ile gelen `%w` sarmalama ve `errors.Is` / `errors.As` fonksiyonları, hata zincirlerinin güvenli şekilde incelenmesini sağlar. Hataları son dönüş değeri olarak kullanmak, bağlam eklerken `%w` tercih etmek ve `errors.Is` / `errors.As` ile kontrol etmek, sürdürülebilir Go kodunun temelini oluşturur.
