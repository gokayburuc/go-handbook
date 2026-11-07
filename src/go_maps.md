# GO - Maps

## Maps Nedir?

Go dilinde `map`, anahtar-değer (key-value) çiftlerini depolayan yerleşik bir hash tablosu veri yapısıdır . Her anahtar benzersiz olmalıdır ve bir anahtara karşılık gelen değere hızlı erişim sağlar. Go'da `map` referans türüdür; yani bir `map` değişkeni, altındaki hash tablosuna işaret eden bir referanstır .

`map`'ler özellikle ilişkisel verileri (kullanıcı ID'sinden kullanıcı bilgisine, ülke kodundan şehirlere, kelimeden frekansa) modellemek için idealdir. Go'nun `map`'i, arama, ekleme ve silme işlemlerinde ortalama O(1) karmaşıklık sunar .

## Sözdizimi (Syntax)

Bir `map` türü şu şekilde tanımlanır:

```go
map[AnahtarTipi]DeğerTipi
```

Örnek: `string` anahtarlardan `int` değerlere bir map.

```go
var m map[string]int // nil map (henüz başlatılmadı)
```

`make` fonksiyonu ile başlatılır:

```go
m := make(map[string]int)
```

Map literal ile de başlatılabilir:

```go
m := map[string]int{
    "bir": 1,
    "iki": 2,
}
```

Anahtar tipi **karşılaştırılabilir** olmalıdır: sayılar, string, bool, pointer, kanal, interface ve karşılaştırılabilir alanlardan oluşan struct'lar kullanılabilir. Slice, map ve fonksiyonlar anahtar olarak **kullanılamaz** .

## Kullanım Alanları

- **Önbellek ve lookup tabloları**: Bir anahtara hızlı erişim gerektiren her senaryo.
- **Sayaçlar ve frekans analizi**: Kelime sayımı, ziyaretçi sayacı gibi işlemler.
- **Küme (set) benzeri yapılar**: `map[string]bool` ile bir elemanın varlığını kontrol etmek .
- **Gruplama**: Aynı kategoriye ait verileri bir arada tutmak (örneğin, ülkeye göre kullanıcılar).
- **Yapılandırma ve kayıt tabloları**: Dinamik olarak büyüyen anahtar-değer koleksiyonları.

## Kullanım Türleri

**1. Ekleme ve Güncelleme**

```go
m := make(map[string]int)
m["elma"] = 5    // ekleme
m["elma"] = 10   // güncelleme (üzerine yazar)
```

**2. Erişim ve Varlık Kontrolü**

Tek değerli erişim, anahtar yoksa **sıfır değeri** döner. İki değerli form, anahtarın var olup olmadığını `bool` olarak bildirir .

```go
v := m["elma"]          // değer veya sıfır değeri
v, ok := m["elma"]      // v = değer, ok = var mı?
```

**3. Silme**

`delete` fonksiyonu, anahtar yoksa hiçbir şey yapmaz ve panic oluşturmaz .

```go
delete(m, "elma")
```

**4. Döngü (range)**

`range` ile anahtar-değer çiftleri üzerinde gezilinir. **Dikkat**: Go'da map iterasyon sırası **rastgeledir**; her çalıştırmada farklı sıra görülebilir .

```go
for k, v := range m {
    fmt.Println(k, v)
}
```

**5. Nil Map Davranışı**

Nil bir map'e **yazmak panic**'e yol açar: `panic: assignment to entry in nil map`. Ancak nil map'ten okumak, silmek veya `range` ile gezmek güvenlidir .

**6. Struct Anahtar Kullanımı**

Birden fazla boyutta gruplama yaparken, iç içe map'ler yerine struct anahtar kullanmak daha temizdir .

```go
type Anahtar struct {
    Yol, Ulke string
}
hits := make(map[Anahtar]int)
hits[Anahtar{"/doc/", "tr"}]++
```

## Örnek Kod

Aşağıdaki örnek, bir e-ticaret sisteminde ürün stoklarını ve kategori bazlı özetleri yöneten gerçekçi bir senaryoyu göstermektedir.

```go
package main

import (
	"fmt"
	"sort"
)

// Urun, bir ürünün temel bilgilerini tutar.
type Urun struct {
	Ad    string
	Fiyat float64
}

func main() {
	// Anahtar: ürün kodu (string), Değer: Urun struct'ı
	// Başlangıç kapasitesi belirtmek, büyük map'lerde yeniden boyutlandırmayı azaltır.
	stok := make(map[string]Urun, 4)

	// Ürün ekleme
	stok["P001"] = Urun{Ad: "Klavye", Fiyat: 450.0}
	stok["P002"] = Urun{Ad: "Fare", Fiyat: 200.0}
	stok["P003"] = Urun{Ad: "Monitör", Fiyat: 3500.0}

	// Güncelleme: aynı anahtar üzerine yazar
	stok["P002"] = Urun{Ad: "Fare (Kablosuz)", Fiyat: 350.0}

	// Erişim ve varlık kontrolü
	if urun, ok := stok["P003"]; ok {
		fmt.Printf("Bulundu: %s, Fiyat: %.2f TL\n", urun.Ad, urun.Fiyat)
	}

	// Olmayan bir anahtara erişim: sıfır değeri döner (burada boş struct)
	olmayan := stok["P999"]
	fmt.Printf("Olmayan ürün adı: '%s'\n", olmayan.Ad) // boş string

	// Silme
	delete(stok, "P001")

	// Map üzerinde döngü: sıra rastgele olduğu için anahtarları toplayıp sıralıyoruz.
	anahtarlar := make([]string, 0, len(stok))
	for kod := range stok {
		anahtarlar = append(anahtarlar, kod)
	}
	sort.Strings(anahtarlar)

	fmt.Println("\nKalan ürünler (sıralı):")
	for _, kod := range anahtarlar {
		u := stok[kod]
		fmt.Printf("  %s -> %s (%.2f TL)\n", kod, u.Ad, u.Fiyat)
	}

	// Kategori bazlı gruplama: map[string][]string
	kategoriler := make(map[string][]string)
	// Anahtar yoksa nil slice'a append güvenlidir; ayrıca var kontrolü gerekmez.
	kategoriler["Elektronik"] = append(kategoriler["Elektronik"], "Klavye", "Fare")
	kategoriler["Ekran"] = append(kategoriler["Ekran"], "Monitör")

	fmt.Println("\nKategoriler:")
	for kat, urunler := range kategoriler {
		fmt.Printf("  %s: %v\n", kat, urunler)
	}

	// Map uzunluğu
	fmt.Printf("\nToplam stok kalemi: %d\n", len(stok))
}
```

**Çıktı:**

```
Bulundu: Monitör, Fiyat: 3500.00 TL
Olmayan ürün adı: ''

Kalan ürünler (sıralı):
  P002 -> Fare (Kablosuz) (350.00 TL)
  P003 -> Monitör (3500.00 TL)

Kategoriler:
  Elektronik: [Klavye Fare]
  Ekran: [Monitör]

Toplam stok kalemi: 2
```

## En İyi Uygulamalar

- **Boyut biliniyorsa önceden kapasite verin.** `make(map[K]V, n)` kullanmak, büyük map'lerde yeniden hash işlemlerini azaltır .
- **Varlık kontrolü için iki değerli formu kullanın.** Sıfır değeri ile "anahtar yok" durumunu karıştırmamak için `_, ok := m[k]` tercih edin .
- **Sıralı çıktı gerekiyorsa anahtarları ayrıca sıralayın.** `range` sırası garanti değildir; `sort.Strings` veya `sort.Ints` ile anahtar dilimini sıralayın .
- **Büyük struct'lar için pointer değer kullanmayı değerlendirin.** Kopyalama maliyetini düşürür; ancak GC baskısını artırabilir .
- **Eşzamanlı erişim gerekiyorsa `sync.RWMutex` veya `sync.Map` kullanın.** Yerleşik map eşzamanlı yazma için güvenli değildir .

> **Sık Yapılan Hatalar ve Uyarılar**
>
> **Nil map'e yazmak panic'e yol açar.** `var m map[string]int` ile tanımlanan bir map başlatılmadan kullanılırsa `panic: assignment to entry in nil map` hatası alınır. Okuma, silme ve `range` nil map'te güvenlidir; yalnızca yazma panik yapar . **Yerleşik map eşzamanlı kullanıma uygun değildir.** Birden fazla goroutine aynı map'e aynı anda yazarsa veya biri yazarken diğeri okursa, Go çalışma zamanı `fatal error: concurrent map read and map write` ile süreci sonlandırır; bu hata `recover` ile yakalanamaz . **Silme öncesi varlık kontrolü gereksizdir.** `delete(m, k)` anahtar yoksa hiçbir şey yapmaz; `if _, ok := m[k]; ok { delete(m, k) }` yazmaya gerek yoktur . **Map değerini fonksiyona kopyalamak altındaki tabloyu kopyalamaz.** Bir struct içindeki map alanı fonksiyona değer olarak geçse bile, fonksiyon içinde yapılan değişiklikler aynı alt tabloyu etkiler .

## Kaynakça

- Go Resmi Blog – Go maps in action: https://go.dev/blog/maps
- Go Dil Spesifikasyonu – Map types: https://go.dev/ref/spec#Map_types
- Go Paket Dokümantasyonu – `maps` paketi (Go 1.21+): https://pkg.go.dev/maps
- Go Paket Dokümantasyonu – `sync` (RWMutex, Map): https://pkg.go.dev/sync

## Sonuç

Go'da `map`, anahtar-değer verilerini yönetmek için güçlü ve verimli bir yerleşik veri yapısıdır. Referans türü olması, nil map davranışı ve rastgele iterasyon sırası gibi özellikleri başlangıçta dikkat gerektirir. Doğru başlatma (`make` veya literal), varlık kontrolü için iki değerli form ve eşzamanlı erişimde uygun senkronizasyon mekanizmaları kullanıldığında `map`, Go programlarının vazgeçilmez bir parçası haline gelir.
