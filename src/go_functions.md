# GO - Functions

## Function Nedir?

Go dilinde fonksiyon, belirli bir işi gerçekleştiren ve yeniden kullanılabilen kod bloğudur. Go'da fonksiyonlar **birinci sınıf vatandaş** (first-class citizens) olarak kabul edilir; yani değişkenlere atanabilir, başka fonksiyonlara argüman olarak geçirilebilir ve fonksiyonlardan döndürülebilir . Her Go programı en az bir fonksiyon içerir: `main` paketindeki `main` fonksiyonu, programın giriş noktasıdır.

Go fonksiyonlarının öne çıkan özellikleri şunlardır: çoklu değer döndürme, isimlendirilmiş dönüş değerleri, değişken argümanlar ve anonim fonksiyonlar (closure) desteği .

## Sözdizimi (Syntax)

Bir fonksiyon tanımlamak için `func` anahtar kelimesi kullanılır:

```go
func fonksiyonAdi(parametre1 tip1, parametre2 tip2) donusTipi {
    // fonksiyon gövdesi
    return deger
}
```

Birden fazla değer döndürmek için dönüş tipleri parantez içinde belirtilir:

```go
func bolVeKalan(a, b int) (int, int) {
    return a / b, a % b
}
```

İsimlendirilmiş dönüş değerleri kullanıldığında, `return` ifadesi çıplak olarak yazılabilir:

```go
func hesapla(x int) (kare, kup int) {
    kare = x * x
    kup = x * x * x
    return // kare ve kup otomatik döner
}
```

## Kullanım Alanları

- **Kod tekrarını önleme**: Aynı mantığı birden fazla yerde yazmak yerine fonksiyon olarak tanımlamak.
- **Soyutlama**: Karmaşık işlemleri anlamlı isimlerle paketleyerek okunabilirliği artırmak.
- **Hata yönetimi**: Go'da fonksiyonlar genellikle ikinci dönüş değeri olarak `error` döndürür.
- **Geri çağırma (callback) ve yüksek mertebeli fonksiyonlar**: Fonksiyonları argüman olarak alan veya döndüren fonksiyonlar yazmak.
- **Durum kapsülleme**: Closure'lar aracılığıyla yerel değişkenleri hapsetmek.

## Kullanım Türleri

**1. Temel Fonksiyon**: Parametre alan ve tek değer döndüren standart fonksiyon.

```go
func topla(a, b int) int {
    return a + b
}
```

**2. Çoklu Dönüş Değeri**: Go'nun en yaygın hata yönetimi kalıbı.

```go
func bol(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("sıfıra bölme hatası")
    }
    return a / b, nil
}
```

**3. İsimlendirilmiş Dönüş Değerleri**: Dönüş değişkenleri fonksiyon imzasında tanımlanır ve sıfır değerleriyle başlatılır.

```go
func minMax(x []int) (min, max int) {
    min, max = x[0], x[0]
    for _, v := range x {
        if v < min { min = v }
        if v > max { max = v }
    }
    return
}
```

**4. Anonim Fonksiyon ve Closure**: İsimsiz fonksiyonlar bir değişkene atanabilir veya doğrudan çağrılabilir. Dış kapsamdaki değişkenleri yakalayabilir .

```go
sayac := 0
artir := func() int {
    sayac++ // dış değişkeni yakalar
    return sayac
}
```

**5. Yüksek Mertebeli Fonksiyon**: Fonksiyonu parametre olarak alan veya döndüren fonksiyon.

```go
func uygula(f func(int) int, x int) int {
    return f(x)
}
```

**6. Defer**: Fonksiyon çıkışında çalıştırılacak işlemleri erteler; kaynak temizliği için kullanılır.

```go
func dosyaOku(yol string) error {
    f, err := os.Open(yol)
    if err != nil { return err }
    defer f.Close() // fonksiyon bitince kapanır
    // ...
    return nil
}
```

## Örnek Kod

Aşağıdaki örnek, bir kullanıcı kayıt sisteminde doğrulama ve raporlama işlemlerini fonksiyonlar aracılığıyla gerçekleştiren gerçekçi bir senaryoyu göstermektedir.

```go
package main

import (
	"errors"
	"fmt"
	"strings"
)

// Kullanici, sistemdeki bir kullanıcıyı temsil eder.
type Kullanici struct {
	Ad    string
	Email string
	Yas   int
}

// Dogrula, kullanıcı bilgilerini kontrol eder ve hata varsa döner.
// Çoklu dönüş değeri ve isimlendirilmiş dönüş kullanımını gösterir.
func Dogrula(k Kullanici) (gecerli bool, hata error) {
	if strings.TrimSpace(k.Ad) == "" {
		return false, errors.New("ad alanı boş olamaz")
	}
	if !strings.Contains(k.Email, "@") {
		return false, errors.New("geçersiz e-posta adresi")
	}
	if k.Yas < 18 {
		return false, errors.New("kullanıcı 18 yaşından küçük olamaz")
	}
	return true, nil
}

// RaporOlustur, geçerli kullanıcılar için özet metni üretir.
// Değişken argümanlı fonksiyon kullanımını gösterir.
func RaporOlustur(kullanicilar ...Kullanici) string {
	var sb strings.Builder
	sb.WriteString("Kullanıcı Raporu\n")
	sb.WriteString("----------------\n")

	// Anonim fonksiyon ve closure: geçerli sayısını saymak için
	gecerliSay := 0
	say := func() {
		gecerliSay++
	}

	for _, k := range kullanicilar {
		ok, err := Dogrula(k)
		if ok {
			say()
			fmt.Fprintf(&sb, "[GECERLI] %s (%s)\n", k.Ad, k.Email)
		} else {
			fmt.Fprintf(&sb, "[HATALI] %s: %v\n", k.Ad, err)
		}
	}

	fmt.Fprintf(&sb, "----------------\nToplam geçerli: %d\n", gecerliSay)
	return sb.String()
}

// Filtrele, bir fonksiyonu parametre olarak alan yüksek mertebeli fonksiyon örneğidir.
func Filtrele(kullanicilar []Kullanici, kosul func(Kullanici) bool) []Kullanici {
	var sonuc []Kullanici
	for _, k := range kullanicilar {
		if kosul(k) {
			sonuc = append(sonuc, k)
		}
	}
	return sonuc
}

func main() {
	kullanicilar := []Kullanici{
		{Ad: "Ayşe", Email: "ayse@ornek.com", Yas: 25},
		{Ad: "Mehmet", Email: "mehmet-ornek.com", Yas: 30},
		{Ad: "Zeynep", Email: "zeynep@ornek.com", Yas: 16},
		{Ad: "Ali", Email: "ali@ornek.com", Yas: 22},
	}

	// Rapor oluştur (değişken argümanlı fonksiyona slice spread ile iletiliyor)
	rapor := RaporOlustur(kullanicilar...)
	fmt.Println(rapor)

	// Yüksek mertebeli fonksiyon: 18 yaşından büyükleri filtrele
	yetiskinler := Filtrele(kullanicilar, func(k Kullanici) bool {
		return k.Yas >= 18
	})
	fmt.Printf("Yetişkin kullanıcı sayısı: %d\n", len(yetiskinler))

	// Çoklu dönüş değeri: doğrudan hata kontrolü
	_, err := Dogrula(Kullanici{Ad: "Test", Email: "test@test.com", Yas: 15})
	if err != nil {
		fmt.Printf("Doğrulama hatası: %v\n", err)
	}
}
```

**Çıktı:**

```
Kullanıcı Raporu
----------------
[GECERLI] Ayşe (ayse@ornek.com)
[HATALI] Mehmet: geçersiz e-posta adresi
[HATALI] Zeynep: kullanıcı 18 yaşından küçük olamaz
[GECERLI] Ali (ali@ornek.com)
----------------
Toplam geçerli: 2

Yetişkin kullanıcı sayısı: 3
Doğrulama hatası: kullanıcı 18 yaşından küçük olamaz
```

## En İyi Uygulamalar

- **Fonksiyonları küçük ve tek amaçlı tutun.** Bir fonksiyon yalnızca bir iş yapmalıdır; isminde "ve" geçiyorsa bölünmesi gerekebilir.
- **Hata döndüren fonksiyonlarda `error`'ı son dönüş değeri yapın.** Go'nun yaygın kalıbı budur; çağıran taraf `if err != nil` ile hemen kontrol eder .
- **Dışa açılan fonksiyonlar için dokümantasyon yorumu yazın.** Yorum, fonksiyon adıyla başlamalı ve ne yaptığını tek cümlede özetlemelidir .
- **Büyük struct'ları pointer ile geçirin.** Go'da parametreler değer olarak kopyalanır; büyük veri yapılarında pointer kullanmak kopyalama maliyetini düşürür .
- **Closure'larda döngü değişkenlerine dikkat edin.** Goroutine içinde closure kullanırken döngü değişkenini parametre olarak geçirin; aksi halde tüm goroutine'ler aynı değişkenin son değerini görebilir .

> **Sık Yapılan Hatalar ve Uyarılar**
>
> Go'da **kullanılmayan değişkenler derleme hatasına yol açar**; geçici olarak kullanılmayan değerler için boş tanımlayıcı `_` kullanılabilir. **İsimlendirilmiş dönüş değerleri** ile çıplak `return` kullanıldığında, dönüş değerleri fonksiyonun başında sıfır değeriyle başlatılır; bu davranış bazen beklenmedik sonuçlara yol açabilir. **Closure'lar dış değişkenleri referans olarak yakalar**; döngü içinde goroutine başlatırken bu durum yarış koşuluna (race condition) neden olabilir . Go'da **kayan nokta sayıları üzerinde fonksiyon çağrısı yaparken dönüş değerini kontrol etmek** önemlidir; özellikle hata döndüren fonksiyonlarda `_` ile hatayı yutmak hata ayıklamayı zorlaştırır.

## Kaynakça

- Go Dil Spesifikasyonu – Function declarations: https://go.dev/ref/spec#Function_declarations
- Go by Example – Functions: https://gobyexample.com/functions
- Effective Go – Functions: https://go.dev/doc/effective_go#functions
- Go Wiki – Common Mistakes (Closures): https://go.dev/wiki/CommonMistakes

## Sonuç

Go'da fonksiyonlar, dilin temel yapı taşlarıdır ve birinci sınıf vatandaş olarak esnek kullanım sunar. Çoklu dönüş değerleri, isimlendirilmiş dönüş parametreleri, değişken argümanlar ve closure desteğiyle hem basit yardımcı fonksiyonlar hem de karmaşık iş mantıkları temiz bir şekilde ifade edilebilir. Hata yönetiminde `error`'ı son dönüş değeri olarak kullanmak Go'nun yerleşik kalıbıdır. Fonksiyonları küçük, odaklı ve iyi belgelenmiş tutmak, sürdürülebilir Go kodunun temelini oluşturur.
