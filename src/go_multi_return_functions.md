# GO - Multi Return Functions

## Multi Return Function Nedir?

Go dilinde bir fonksiyon, tek bir değer yerine **birden fazla değer döndürebilir**. Bu özellik Go'nun yerleşik desteğidir ve dilin ayırt edici özelliklerinden biridir . Çoklu dönüş değerleri, özellikle hata yönetiminde olmak üzere, Go'nun deyimsel (idiomatic) kod yazımının temel bir parçasıdır . Bir fonksiyon hem sonucu hem de bir hata durumunu aynı anda döndürebilir; bu sayede çağıran taraf her iki bilgiye de tek çağrıda erişir.

## Sözdizimi (Syntax)

Çoklu dönüş değerleri, fonksiyon imzasında dönüş tiplerinin parantez içinde virgülle ayrılarak belirtilmesiyle tanımlanır:

```go
func fonksiyonAdi(parametreler) (donusTipi1, donusTipi2, ...) {
    return deger1, deger2, ...
}
```

Tek dönüş değeri olduğunda parantez gerekmezken, birden fazla olduğunda tipler parantez içine alınır .

```go
// İki tam sayı döndüren fonksiyon
func vals() (int, int) {
    return 3, 7
}
```

Dönüş değerleri isteğe bağlı olarak **isimlendirilebilir**. Bu durumda `return` ifadesi çıplak (naked return) yazılabilir:

```go
func bolVeKalan(a, b int) (bolum, kalan int) {
    bolum = a / b
    kalan = a % b
    return // bolum ve kalan otomatik döner
}
```

## Kullanım Alanları

Çoklu dönüş değerleri özellikle şu durumlarda kullanılır :

- **Hata yönetimi**: Go'da istisna (exception) mekanizması yoktur; bunun yerine `(sonuc, error)` kalıbı kullanılır. Bu, Go'nun en yaygın çoklu dönüş değeri kullanımıdır.
- **Başarı sinyali**: Bir arama veya lookup işleminde hem değer hem de `bool` dönmek (örneğin `(deger, bulundu)`).
- **İlişkili değerler**: Bir işlemin doğal olarak ürettiği iki değeri (bölüm ve kalan, minimum ve maksimum, enlem ve boylam) tek çağrıda döndürmek.
- **Durum ve ek bağlam**: Bir sonuçla birlikte ek bilgi (örneğin, işlenen öğe sayısı) döndürmek.

## Kullanım Türleri

**1. Temel Çoklu Dönüş**: İki veya daha fazla değer döndürme.

```go
func koordinat() (float64, float64) {
    return 41.0082, 28.9784
}
```

**2. Hata ile Birlikte Dönüş**: Go'nun standart hata yönetimi kalıbı. Hata her zaman **son dönüş değeri** olarak yer alır .

```go
func guvenliBol(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("sıfıra bölme hatası")
    }
    return a / b, nil
}
```

**3. Boolean ile Birlikte Dönüş**: Bir değerin var olup olmadığını bildirmek için.

```go
func kullaniciBul(id int) (Kullanici, bool) {
    k, varMi := kullaniciMap[id]
    return k, varMi
}
```

**4. İsimlendirilmiş Dönüş Değerleri**: Dönüş değişkenleri imzada tanımlanır ve sıfır değerleriyle başlatılır. Çıplak `return` ile döndürülür .

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

**5. Boş Tanımlayıcı ile Değer Atlama**: Dönen değerlerden yalnızca bir kısmı gerekiyorsa, istenmeyen değerler `_` ile atılır .

```go
sonuc, _ := guvenliBol(10, 2) // hata yok sayılıyor (önerilmez)
_, kalan := bolVeKalan(10, 3) // sadece kalan isteniyor
```

## Örnek Kod

Aşağıdaki örnek, bir dosya işleme ve kullanıcı doğrulama senaryosunda çoklu dönüş değerlerinin pratik kullanımını göstermektedir.

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
}

// kullaniciVeritabani, örnek bir kullanıcı deposunu simüle eder.
var kullaniciVeritabani = map[int]Kullanici{
	1: {Ad: "Ayşe", Email: "ayse@ornek.com"},
	2: {Ad: "Mehmet", Email: "mehmet@ornek.com"},
}

// KullaniciBul, verilen ID'ye karşılık gelen kullanıcıyı arar.
// İki değer döndürür: bulunan kullanıcı ve bulunup bulunmadığını belirten bool.
func KullaniciBul(id int) (Kullanici, bool) {
	k, varMi := kullaniciVeritabani[id]
	return k, varMi
}

// EmailDogrula, bir e-posta adresinin geçerli olup olmadığını kontrol eder.
// Hata durumunda açıklayıcı bir error döndürür.
func EmailDogrula(email string) (bool, error) {
	if strings.TrimSpace(email) == "" {
		return false, errors.New("e-posta adresi boş olamaz")
	}
	if !strings.Contains(email, "@") {
		return false, errors.New("e-posta adresi '@' karakteri içermelidir")
	}
	return true, nil
}

// KullaniciEkle, yeni bir kullanıcı ekler ve eklenen kullanıcının ID'sini döndürür.
// İsimlendirilmiş dönüş değerleri kullanır.
func KullaniciEkle(k Kullanici) (id int, hata error) {
	// E-posta doğrulaması yap
	gecerli, err := EmailDogrula(k.Email)
	if err != nil || !gecerli {
		return 0, err
	}

	// Yeni ID ata
	id = len(kullaniciVeritabani) + 1
	kullaniciVeritabani[id] = k
	return id, nil // açık dönüş
}

// BolVeKalan, iki tam sayının bölümünü ve kalanını döndürür.
// Sıfıra bölme durumunda hata döndürür.
func BolVeKalan(a, b int) (int, int, error) {
	if b == 0 {
		return 0, 0, errors.New("sıfıra bölme yapılamaz")
	}
	return a / b, a % b, nil
}

func main() {
	// 1. Boolean ile birlikte dönüş: kullanıcı arama
	if k, bulundu := KullaniciBul(1); bulundu {
		fmt.Printf("Kullanıcı bulundu: %s (%s)\n", k.Ad, k.Email)
	} else {
		fmt.Println("Kullanıcı bulunamadı")
	}

	// 2. Hata ile birlikte dönüş: e-posta doğrulama
	gecerli, err := EmailDogrula("gecersiz-email")
	if err != nil {
		fmt.Printf("Doğrulama hatası: %v\n", err)
	} else {
		fmt.Printf("E-posta geçerli mi: %t\n", gecerli)
	}

	// 3. İsimlendirilmiş dönüş değerleri: kullanıcı ekleme
	yeniID, err := KullaniciEkle(Kullanici{Ad: "Zeynep", Email: "zeynep@ornek.com"})
	if err != nil {
		fmt.Printf("Ekleme hatası: %v\n", err)
	} else {
		fmt.Printf("Kullanıcı eklendi, ID: %d\n", yeniID)
	}

	// 4. Üç değer dönüşü: bölme işlemi
	bolum, kalan, err := BolVeKalan(17, 5)
	if err != nil {
		fmt.Printf("Bölme hatası: %v\n", err)
	} else {
		fmt.Printf("17 / 5 = %d, kalan = %d\n", bolum, kalan)
	}

	// 5. Boş tanımlayıcı ile değer atlama
	_, sadeceKalan, _ := BolVeKalan(10, 3)
	fmt.Printf("Sadece kalan: %d\n", sadeceKalan)
}
```

**Çıktı:**

```
Kullanıcı bulundu: Ayşe (ayse@ornek.com)
Doğrulama hatası: e-posta adresi '@' karakteri içermelidir
Kullanıcı eklendi, ID: 3
17 / 5 = 3, kalan = 2
Sadece kalan: 1
```

## En İyi Uygulamalar

- **Hata her zaman son dönüş değeri olsun.** Go topluluğunun yerleşik kuralı `(sonuc, error)` şeklindedir; `(error, sonuc)` değil .
- **Hata döndüğünde sıfır değeri döndürün.** `(T, error)` kalıbında, hata `nil` değilse `T` için genellikle sıfır değeri döndürülür; bu, çağıran tarafın yanlışlıkla geçersiz bir değer kullanmasını önler .
- **Dönüş değeri sayısını 2-3 ile sınırlayın.** Daha fazla değer gerekiyorsa, bir struct içinde gruplamak okunabilirliği artırır .
- **Hataları hemen kontrol edin.** Çoklu dönüş değeri alındığında `if err != nil` kontrolü ilk iş olmalıdır .
- **İsimlendirilmiş dönüş değerlerini kısa fonksiyonlarda kullanın.** Uzun fonksiyonlarda çıplak `return`, kodun okunabilirliğini azaltabilir .

> **Sık Yapılan Hatalar ve Uyarılar**
>
> **Boş tanımlayıcı `_` ile hatayı yok saymak** yaygın bir hatadır; `sonuc, _ := riskliFonksiyon()` yazmak, oluşabilecek hataları sessizce yutar ve hata ayıklamayı zorlaştırır. Üretim kodunda hatalar mutlaka ele alınmalıdır. **İsimlendirilmiş dönüş değerleri** kullanıldığında, dönüş değişkenleri fonksiyon başında sıfır değeriyle başlatılır; bu durum bazen beklenmedik sonuçlara yol açabilir. **Çok sayıda dönüş değeri** (4 veya daha fazla), çağıran tarafın hangi değerin ne anlama geldiğini karıştırmasına neden olabilir; bu durumda bir struct döndürmek daha temizdir. **Hata kontrolünü atlayıp doğrudan sonucu kullanmak**, Go'nun hata yönetimi felsefesine aykırıdır ve çalışma zamanı hatalarına davetiye çıkarır.

## Kaynakça

- Go by Example – Multiple Return Values: https://gobyexample.com/multiple-return-values
- Go Dil Spesifikasyonu – Function declarations: https://go.dev/ref/spec#Function_declarations
- Effective Go – Multiple return values: https://go.dev/doc/effective_go#multiple-returns

## Sonuç

Go'da çoklu dönüş değerleri, dilin hata yönetimi ve veri döndürme biçimini şekillendiren temel bir özelliktir. `(sonuc, error)` kalıbı, Go kodunun vazgeçilmez bir parçasıdır. İsimlendirilmiş dönüş değerleri ve boş tanımlayıcı `_`, bu yapıyı daha esnek hale getirir. Dönüş değerlerini 2-3 ile sınırlamak, hataları her zaman son sıraya koymak ve hataları derhal kontrol etmek, okunabilir ve güvenilir Go kodu yazmanın anahtarıdır.
