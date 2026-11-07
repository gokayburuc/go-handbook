# GO - Structs

## Structs Nedir?

Go dilinde `struct`, farklı türlerdeki alanları (field) tek bir çatı altında toplayan **kullanıcı tanımlı bir bileşik türdür** . Go'da sınıf (class) kavramı yoktur; bunun yerine struct'lar, gerçek dünyadaki varlıkları modellemek ve ilişkili verileri bir arada tutmak için kullanılır . Bir struct, sıfır veya daha fazla alan içerebilir ve bu alanlar farklı türlerde olabilir .

Go'da kalıtım (inheritance) yoktur; bunun yerine **kompozisyon** (composition) ve **gömme** (embedding) mekanizmaları kullanılır . Struct alanlarının ilk harfinin büyük veya küçük olması, o alanın paket dışına açık (exported) olup olmadığını belirler .

## Sözdizimi (Syntax)

Bir struct tanımlamak için `type` ve `struct` anahtar kelimeleri kullanılır:

```go
type Kullanici struct {
    Ad    string
    Email string
    Yas   int
}
```

Struct değerleri iki şekilde başlatılabilir: **anahtarlı** (keyed) veya **anahtarsız** (unkeyed) literal.

```go
// Anahtarlı literal (önerilen)
k := Kullanici{Ad: "Ayşe", Email: "ayse@ornek.com", Yas: 25}

// Anahtarsız literal (alan sırasına bağlı, önerilmez)
k := Kullanici{"Ayşe", "ayse@ornek.com", 25}
```

Alanlara `.` (nokta) operatörü ile erişilir: `k.Ad` .

## Kullanım Alanları

- **Gerçek dünya varlıklarını modelleme**: Kullanıcı, sipariş, ürün gibi kayıtları temsil etmek .
- **JSON kodlama/çözme**: `encoding/json` paketi ile struct'lar doğrudan JSON'a dönüştürülebilir; alan etiketleri (tag) ile JSON anahtar isimleri özelleştirilir .
- **Fonksiyon parametrelerini gruplama**: Çok sayıda parametre yerine tek bir struct parametresi kullanmak okunabilirliği artırır .
- **Veri gruplama**: İlişkili alanları tek bir değer olarak taşımak ve fonksiyonlara geçirmek.

## Kullanım Türleri

**1. Temel Struct ve Metotlar**

Struct'lara metot tanımlanabilir. Receiver değer (value) veya pointer olabilir. Değer receiver'ı struct'ın kopyası üzerinde çalışır; pointer receiver'ı orijinali değiştirir .

```go
type Sayac struct {
    Deger int
}

// Değer receiver: kopya üzerinde çalışır
func (s Sayac) Oku() int {
    return s.Deger
}

// Pointer receiver: orijinali değiştirir
func (s *Sayac) Artir() {
    s.Deger++
}
```

**2. Gömme (Embedding)**

Bir struct, başka bir struct'ı isim vermeden içerebilir. İçteki struct'ın alanları ve metotları dıştaki struct'a **terfi eder** (promoted) .

```go
type Kisi struct {
    Ad string
}

type Ogrenci struct {
    Kisi      // gömülü alan (anonymous field)
    Numara int
}

o := Ogrenci{Kisi: Kisi{Ad: "Ali"}, Numara: 100}
fmt.Println(o.Ad) // "Ali" - doğrudan erişilir
```

**3. Anonim Struct**

İsimsiz, tek seferlik struct'lar hızlıca tanımlanabilir .

```go
nokta := struct {
    X, Y int
}{X: 10, Y: 20}
```

**4. Alan Etiketleri (Tags)**

Struct alanlarına metadata eklemek için etiketler kullanılır. En yaygın kullanım JSON serileştirmedir .

```go
type Siparis struct {
    ID    string  `json:"id"`
    Total float64 `json:"total"`
    Not   string  `json:"note,omitempty"` // boşsa JSON'a yazılmaz
}
```

## Örnek Kod

Aşağıdaki örnek, bir kütüphane sisteminde kitap ve üye yönetimini struct'lar aracılığıyla göstermektedir.

```go
package main

import (
	"encoding/json"
	"fmt"
)

// Kitap, kütüphanedeki bir kitabı temsil eder.
type Kitap struct {
	Baslik string  `json:"baslik"`
	Yazar  string  `json:"yazar"`
	Fiyat  float64 `json:"fiyat"`
	Oduncta bool   `json:"oduncta"`
}

// Uye, kütüphane üyesini temsil eder.
type Uye struct {
	Ad     string   `json:"ad"`
	Numara int      `json:"numara"`
	Kitaplar []Kitap `json:"kitaplar"` // iç içe struct dilimi
}

// OduncVer, kitabı ödünç verildi olarak işaretler.
// Pointer receiver kullanır; orijinal kitap değişir.
func (k *Kitap) OduncVer() {
	k.Oduncta = true
}

// IadeEt, kitabı iade alındı olarak işaretler.
func (k *Kitap) IadeEt() {
	k.Oduncta = false
}

// UyeBilgisi, üyenin ödünç aldığı kitap sayısını döndürür.
// Değer receiver kullanır; sadece okuma yapar.
func (u Uye) OduncKitapSayisi() int {
	sayac := 0
	for _, kitap := range u.Kitaplar {
		if kitap.Oduncta {
			sayac++
		}
	}
	return sayac
}

func main() {
	// Kitap oluşturma (anahtarlı literal)
	k1 := Kitap{Baslik: "Go Programlama", Yazar: "A. Yılmaz", Fiyat: 150.0}
	k2 := Kitap{Baslik: "Veri Yapıları", Yazar: "B. Demir", Fiyat: 200.0}

	// Pointer receiver ile metod çağrısı
	k1.OduncVer()

	// Üye oluşturma: iç içe struct dilimi
	uye := Uye{
		Ad:     "Zeynep",
		Numara: 42,
		Kitaplar: []Kitap{k1, k2},
	}

	fmt.Printf("Üye: %s (No: %d)\n", uye.Ad, uye.Numara)
	fmt.Printf("Ödünç alınan kitap sayısı: %d\n", uye.OduncKitapSayisi())

	// JSON'a dönüştürme: alan etiketleri kullanılır
	jsonVeri, _ := json.MarshalIndent(uye, "", "  ")
	fmt.Println("\nJSON çıktısı:")
	fmt.Println(string(jsonVeri))

	// Kitabı iade et ve durumu kontrol et
	k1.IadeEt()
	fmt.Printf("\n'%s' ödünçte mi: %t\n", k1.Baslik, k1.Oduncta)
}
```

**Çıktı:**

```
Üye: Zeynep (No: 42)
Ödünç alınan kitap sayısı: 1

JSON çıktısı:
{
  "ad": "Zeynep",
  "numara": 42,
  "kitaplar": [
    {
      "baslik": "Go Programlama",
      "yazar": "A. Yılmaz",
      "fiyat": 150,
      "oduncta": true
    },
    {
      "baslik": "Veri Yapıları",
      "yazar": "B. Demir",
      "fiyat": 200,
      "oduncta": false
    }
  ]
}

'Go Programlama' ödünçte mi: false
```

## En İyi Uygulamalar

- **Anahtarlı literal kullanın.** `Kullanici{Ad: "Ali"}` yazımı, alan sırasına bağımlılığı ortadan kaldırır ve yeni alan eklendiğinde kodu bozmaz .
- **Bir tür için tek bir receiver türü seçin.** Eğer bir metot pointer receiver gerektiriyorsa, tüm metotlar pointer receiver almalıdır; karışık kullanım tutarsızlık yaratır .
- **Dışa açılan struct alanlarını gerektiğinde etiketleyin.** JSON, XML gibi serileştirme kullanan tüm dışa açık alanlara etiket ekleyin; aksi halde alan adları Go'daki haliyle serileştirilir .
- **Büyük struct'ları pointer ile geçirin.** Go'da parametreler değer olarak kopyalanır; büyük struct'larda pointer kullanmak kopyalama maliyetini düşürür .
- **Gömme yerine isimli alan kullanmayı değerlendirin.** İçteki türün tüm API'sini dışa açmak istemiyorsanız, gömme yerine isimli alan tercih edin .

> **Sık Yapılan Hatalar ve Uyarılar**
>
> **Anahtarsız literal kullanımı** (`Kullanici{"Ali", 25}`), struct tanımına yeni bir alan eklendiğinde veya alan sırası değiştiğinde derleme hatasına veya yanlış atamalara yol açar; `go vet` bu kullanımı uyarı olarak işaretler . **Struct'ı fonksiyona değer olarak geçirmek**, fonksiyon içinde yapılan değişikliklerin orijinal struct'a yansımamasına neden olur; değişiklik gerekiyorsa pointer kullanılmalıdır . **Gömülü alanın sıfır değeri**, dıştaki struct başlatılırken içteki struct'ın da açıkça başlatılmasını gerektirebilir; aksi halde iç alanlar sıfır değerinde kalır. **Dışa açık alanlarda etiket eksikliği**, JSON çıktısında beklenmeyen anahtar isimlerine (örneğin `FirstName` yerine `first_name` beklenirken `FirstName`) yol açar .

## Kaynakça

- Go Dil Spesifikasyonu – Struct types: https://go.dev/ref/spec#Struct_types
- A Tour of Go – Structs: https://go.dev/tour/moretypes/2
- Go Packages – `encoding/json`: https://pkg.go.dev/encoding/json
- Effective Go – Composite literals: https://go.dev/doc/effective_go#composite_literals

## Sonuç

Go'da struct'lar, ilişkili verileri tek bir tür altında toplayan temel yapı taşlarıdır. Sınıf ve kalıtım olmadan, kompozisyon ve gömme yoluyla karmaşık veri modelleri oluşturulur. Metotlar ile struct'lara davranış kazandırılır; pointer receiver'lar orijinal değeri değiştirmek için kullanılır. Alan etiketleri sayesinde JSON gibi formatlarla sorunsuz entegrasyon sağlanır. Anahtarlı literal kullanımı, tek tip receiver seçimi ve büyük struct'larda pointer tercihi, sürdürülebilir Go kodunun temelini oluşturur.
