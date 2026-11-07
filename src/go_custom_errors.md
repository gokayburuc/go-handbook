# GO - Custom Errors

## Custom Errors Nedir?

Go'da özel hata türü (custom error), yerleşik `error` arayüzünü uygulayan kullanıcı tanımlı bir türdür. `error` arayüzü tek bir metot içerir: `Error() string`. Bu metodu tanımlayan her tür, bir hata değeri olarak kullanılabilir . Standart `errors.New` veya `fmt.Errorf` ile oluşturulan hatalar yalnızca bir mesaj metni taşırken, özel hata türleri **yapılandırılmış veri** (alan adı, hata kodu, kaynak kimliği gibi) taşıyabilir. Bu sayede çağıran taraf, hata mesajını metin olarak ayrıştırmak zorunda kalmadan hatanın türüne ve içeriğine programatik olarak erişebilir .

## Sözdizimi (Syntax)

Özel bir hata türü tanımlamak için bir struct oluşturulur ve bu struct'a `Error() string` metodu eklenir. Go geleneğinde özel hata türlerinin adı genellikle **"Error" son ekiyle** biter (`ValidationError`, `NotFoundError` gibi) .

```go
type DogrulamaHatasi struct {
    Alan  string
    Mesaj string
}

// Error metodu, DogrulamaHatasi türünü error arayüzünü uygular hale getirir.
func (e *DogrulamaHatasi) Error() string {
    return fmt.Sprintf("%s: %s", e.Alan, e.Mesaj)
}
```

Fonksiyondan döndürürken pointer (`&DogrulamaHatasi{...}`) veya değer (`DogrulamaHatasi{...}`) kullanılabilir. Pointer kullanımı, `errors.As` ile tür kontrolü yapılırken daha yaygın ve tutarlıdır .

## Kullanım Alanları

- **Alan bazlı doğrulama hataları**: Hangi alanın hangi kuralı ihlal ettiğini yapılandırılmış olarak taşımak (örneğin `ValidationError{Field: "email", Message: "geçersiz"}`) .
- **Kaynak bulunamadı hataları**: Aranan kaynağın türünü ve kimliğini hata içinde saklamak (`NotFoundError{Resource: "user", ID: "42"}`) .
- **HTTP durum kodu veya iş hata kodu taşıma**: API katmanında hatayı uygun HTTP yanıtına dönüştürmek için hata içinde kod bilgisi tutmak .
- **Hata zincirinde tür güvenli inceleme**: Sarmalanmış (wrapped) hatalar arasından belirli bir özel hata türünü çekip almak için `errors.As` kullanmak .

## Kullanım Türleri

**1. Temel Özel Hata ve `errors.As` ile Kontrol**

Özel hata türü, `errors.As` fonksiyonu ile hata zinciri içinden çekilip alınır. `errors.As`, hata sarmalanmış olsa bile doğru türe ulaşmayı sağlar .

```go
var hedef *DogrulamaHatasi
if errors.As(err, &hedef) {
    fmt.Println("Geçersiz alan:", hedef.Alan)
}
```

**2. Hata Sarmalama ile Birlikte Kullanım**

Özel hata, `%w` fiili ile sarmalanarak üst katmana bağlam eklenebilir. `errors.As` bu sarmalamayı aşarak özel türe ulaşır .

```go
return fmt.Errorf("kullanıcı işlenemedi: %w", &DogrulamaHatasi{...})
```

**3. Sentinel Hata ile Karşılaştırma**

Sentinel hata (`var ErrNotFound = errors.New(...)`) tek bir sabit durumu temsil ederken, özel hata türü ek veri taşır. Hata veri taşıması gerekiyorsa özel tür, yalnızca eşleştirme gerekiyorsa sentinel tercih edilir .

## Örnek Kod

Aşağıdaki örnek, bir kullanıcı kayıt sisteminde alan bazlı doğrulama ve kaynak bulunamadı hatalarını özel hata türleriyle modellemektedir.

```go
package main

import (
	"errors"
	"fmt"
)

// DogrulamaHatasi, alan bazli dogrulama hatalarini tasir.
type DogrulamaHatasi struct {
	Alan  string
	Mesaj string
}

// Error metodu, DogrulamaHatasi'ni error arayuzune uygun hale getirir.
func (e *DogrulamaHatasi) Error() string {
	return fmt.Sprintf("doğrulama hatası: %s - %s", e.Alan, e.Mesaj)
}

// BulunamadiHatasi, aranan kaynagin bulunamadigini belirtir.
type BulunamadiHatasi struct {
	Kaynak string
	ID     string
}

func (e *BulunamadiHatasi) Error() string {
	return fmt.Sprintf("%s bulunamadı: %s", e.Kaynak, e.ID)
}

// KullaniciKaydet, gelen verileri dogrular ve hata varsa ozel hata dondurur.
func KullaniciKaydet(ad, email string) error {
	if ad == "" {
		return &DogrulamaHatasi{Alan: "ad", Mesaj: "boş olamaz"}
	}
	if email == "" {
		return &DogrulamaHatasi{Alan: "email", Mesaj: "boş olamaz"}
	}
	return nil
}

// KullaniciBul, ID'ye gore kullanici arar.
func KullaniciBul(id string) error {
	if id != "42" {
		return &BulunamadiHatasi{Kaynak: "kullanıcı", ID: id}
	}
	return nil
}

func main() {
	// 1. Dogrulama hatasi: alan bilgisine errors.As ile erisim
	err := KullaniciKaydet("", "test@ornek.com")
	if err != nil {
		var dh *DogrulamaHatasi
		if errors.As(err, &dh) {
			fmt.Printf("Hata: %v\n", dh)
			fmt.Printf("  -> Gecersiz alan: %s\n", dh.Alan)
		}
	}

	// 2. Bulunamadi hatasi: kaynak ve ID bilgisine erisim
	err = KullaniciBul("999")
	if err != nil {
		var bh *BulunamadiHatasi
		if errors.As(err, &bh) {
			fmt.Printf("Hata: %v\n", bh)
			fmt.Printf("  -> Kaynak: %s, ID: %s\n", bh.Kaynak, bh.ID)
		}
	}

	// 3. Sarmalanmis hata icinden ozel ture erisim
	sarmalanmis := fmt.Errorf("kayıt işlemi başarısız: %w", &DogrulamaHatasi{Alan: "email", Mesaj: "geçersiz format"})
	var hedef *DogrulamaHatasi
	if errors.As(sarmalanmis, &hedef) {
		fmt.Printf("Sarmalanmış hatadan çıkarıldı: alan=%s\n", hedef.Alan)
	}
}
```

**Çıktı:**

```
Hata: doğrulama hatası: ad - boş olamaz
  -> Gecersiz alan: ad
Hata: kullanıcı bulunamadı: 999
  -> Kaynak: kullanıcı, ID: 999
Sarmalanmış hatadan çıkarıldı: alan=email
```

## En İyi Uygulamalar

- **Hata türü adlarında "Error" son ekini kullanın.** `ValidationError`, `NotFoundError` gibi isimler Go topluluğunda yerleşik bir gelenektir .
- **`errors.As` kullanın, tür assertion yerine.** Sarmalanmış hatalarda `err.(*DogrulamaHatasi)` başarısız olabilir; `errors.As` hata zincirini aşarak doğru türe ulaşır .
- **Özel hata türünü pointer olarak döndürün.** `&DogrulamaHatasi{...}` kullanımı, `errors.As` ile tür kontrolünde tutarlılık sağlar ve metot seti sorunlarını önler .
- **`Error()` metodu kısa ve okunabilir olsun.** Hata mesajları küçük harfle başlamalı ve nokta ile bitmemelidir; mesajlar sarmalandığında cümle ortasında kullanılabilir .
- **Yalnızca veri taşıması gerektiğinde özel tür kullanın.** Sabit bir durumu temsil eden hatalar için sentinel (`errors.New`) yeterlidir; gereksiz struct tanımlamaktan kaçının .

> **Sık Yapılan Hatalar ve Uyarılar**
>
> **`Error()` metodunu değer alıcısıyla tanımlayıp pointer döndürmek** karışıklığa yol açabilir; tutarlılık için pointer alıcısı ve pointer dönüşü tercih edin. **`errors.As` yerine tür assertion kullanmak**, sarmalanmış hatalarda başarısız olur ve hatanın asıl türüne ulaşılamaz . **Özel hata türünü `Error()` metodu olmadan tanımlamak** derleme hatası verir; tür, `error` arayüzünü uygulamaz. **Hata mesajına hassas veri koymak** (şifre, token gibi) log ve kullanıcı çıktısında güvenlik açığı oluşturur; hassas bilgiler hata mesajından ayrı tutulmalıdır.

## Kaynakça

- Go by Example – Custom Errors: https://gobyexample.com/custom-errors
- Go Blog – Working with Errors in Go 1.13: https://go.dev/blog/go1.13-errors
- Go Paket Dokümantasyonu – `errors`: https://pkg.go.dev/errors

## Sonuç

Go'da özel hata türleri, `error` arayüzünü uygulayan struct'lar aracılığıyla hatalara yapılandırılmış veri kazandırır. `Error()` metodu ile arayüz uyumluluğu sağlanır; `errors.As` ile hata zinciri içinden tür güvenli erişim yapılır. Alan bazlı doğrulama, kaynak kimliği taşıma ve HTTP kod eşleştirme gibi senaryolarda standart hata mesajlarından çok daha fazlasını sunar. Hata türü adlarında "Error" son eki, pointer dönüşü ve `errors.As` kullanımı, bu yapının güvenli ve sürdürülebilir kullanımının temelini oluşturur.
