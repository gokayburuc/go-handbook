# 🐹 Go Handbook (Golang El Kitabı)

[![Go](https://img.shields.io/badge/Go-1.22+-00ADD8?style=flat&logo=go&logoColor=white)](https://go.dev/)
[![mdBook](https://img.shields.io/badge/Built%20with-mdBook-FF6B6B?style=flat&logo=rust&logoColor=white)](https://rust-lang.github.io/mdBook/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Kapsamlı, pratik ve modern Go (Golang) başvuru rehberi. Temel sözdiziminden eşzamanlılığa (concurrency), genel türlerden (generics) standart kütüphanelere kadar detaylı açıklamalar ve örnekler içerir.

> An open-source, comprehensive Go programming language handbook and cookbook built with [mdBook](https://rust-lang.github.io/mdBook/). Written primarily in Turkish with clear explanations, syntax breakdowns, and production-ready code snippets.

---

## 📖 İçindekiler (Table of Contents)

- [Hakkında (About)](#-hakkında-about)
- [Konu Başlıkları (Topics Covered)](#-konu-başlıkları-topics-covered)
- [Gereksinimler (Prerequisites)](#-gereksinimler-prerequisites)
- [Kurulum ve Yerel Geliştirme (Getting Started)](#-kurulum-ve-yerel-geliştirme-getting-started)
  - [1. Projeyi Klonlayın](#1-projeyi-klonlayın)
  - [2. Canlı Önizleme (Canlı Sunucu)](#2-canlı-önizleme-canlı-sunucu)
  - [3. Statik HTML Derleme](#3-statik-html-derleme)
- [Dizin Yapısı (Directory Structure)](#-dizin-yapısı-directory-structure)
- [Katkıda Bulunma (Contributing)](#-katkıda-bulunma-contributing)
- [Yazar & İletişim (Author & Contact)](#-yazar--iletişim-author--contact)
- [Lisans (License)](#-lisans-license)

---

## 🎯 Hakkında (About)

Bu rehber, Go dilini öğrenmek isteyenler veya bildiklerini pekiştirmek isteyen geliştiriciler için modüler, anlaşılır ve konuya odaklı bir referans kaynağı olarak hazırlanmıştır.

- **Konu Anlatımları:** Temel ve ileri düzey kavramların sade bir dille izahı.
- **Sözdizimi ve Örnekler:** Her başlık için uygulanabilir ve modern Go pratiklerine uygun kod parçaları.
- **Modern Özellikler:** Go Generics, Range-over-iterators ve modern hata yönetimi desenleri.
- **Standart Kütüphane İncelemeleri:** Go'nun zengin standart kütüphanesinin pratik kullanımı.

---

## 📚 Konu Başlıkları (Topics Covered)

Kitap iki ana eksen etrafında yapılandırılmıştır:

### 1. GO Temelleri (Fundamentals & Core)

- **Değerler, Değişkenler ve Sabitler:** `Values`, `Variables`, `Constants`
- **Kontrol Yapıları:** `Conditionals (if/else)`, `For Loops`, `Switch`
- **Veri Yapıları:** `Arrays`, `Slices`, `Maps`
- **Fonksiyonlar:** `Functions`, `Multiple Returns`, `Variadic Functions`, `Closures`, `Recursion`
- **Nesne Yönelim ve Tipler:** `Pointers`, `Structs`, `Struct Embedding`, `Methods`, `Interfaces`, `Enums`
- **İleri Seviye Özellikler:** `Generics`, `Range over Iterators`
- **Hata Yönetimi:** `Errors`, `Custom Errors`
- **Eşzamanlılık (Concurrency):** `Goroutines`, `Channels`

### 2. Standart Kütüphane (Standard Library)

- **Girdi / Çıktı:** `bufio` kütüphanesi ve pratik dosya / stream okuma-yazma işlemleri

---

## 🛠 Gereksinimler (Prerequisites)

Rehberi yerel makinenizde görüntülemek veya yeni içerik eklemek için aşağıdaki araçlara ihtiyacınız vardır:

- **[mdBook](https://rust-lang.github.io/mdBook/cli/index.html)** (Rust tabanlı hızlı dokümantasyon oluşturucu)
- _(İsteğe bağlı)_ **[Go](https://go.dev/dl/)** (Örnek kodları çalıştırmak için `1.22+`)

### mdBook Kurulumu

**Cargo ile (Rust kurulu ise):**

```bash
cargo install mdbook
```

**Homebrew (macOS / Linux):**

```bash
brew install mdbook
```

Alternatif olarak [mdBook GitHub Releases](https://github.com/rust-lang/mdBook/releases) sayfasından işletim sisteminize uygun ikili dosyayı indirebilirsiniz.

---

## 🚀 Kurulum ve Yerel Geliştirme (Getting Started)

### 1. Projeyi Klonlayın

```bash
git clone git@github.com:gokayburuc/go-handbook.git
cd go-handbook
```

### 2. Canlı Önizleme (Canlı Sunucu)

Yerel bir web sunucusu başlatarak dosyaları değiştirdiğinizde otomatik yenilenen (live reload) önizleme modunu açabilirsiniz:

```bash
mdbook serve --open
```

Tarayıcınız otomatik açılmazsa: [http://localhost:3000](http://localhost:3000) adresini ziyaret edebilirsiniz.

### 3. Statik HTML Derleme

Kitabı statik web sitesi olarak derlemek için:

```bash
mdbook build
```

Oluşturulan statik web sitesi çıktıları `book/` dizini altına kaydedilir. Bu çıktıları GitHub Pages, Netlify, Vercel veya herhangi bir statik dosya sunucusunda doğrudan barındırabilirsiniz.

---

## 📁 Dizin Yapısı (Directory Structure)

```text
.
├── book.toml         # mdBook konfigürasyon dosyası (başlık, yazar, dil vb.)
├── .gitignore        # Git hariç tutma kuralları (derlenen 'book/' dizini hariç)
├── src/              # Markdown formatındaki kitap içerikleri
│   ├── SUMMARY.md    # Kitap içindekiler tablosu ve gezinme haritası
│   ├── go_values.md
│   ├── go_variables.md
│   ├── go_goroutines.md
│   ├── go_channels.md
│   └── ...
└── README.md         # Proje tanıtım ve rehber dokümantasyonu
```

---

## 🤝 Katkıda Bulunma (Contributing)

Katkılarınızı memnuniyetle kabul ediyoruz! Bir yazım hatası bulursanız, yeni bir konu eklemek isterseniz veya örnekleri geliştirmek isterseniz:

1. Bu depoyu Fork'layın (`Fork`).
2. Yeni bir özellik dalı oluşturun (`git checkout -b feature/yeni-konu`).
3. Değişikliklerinizi yapın ve `SUMMARY.md` dosyasına bağlantısını ekleyin.
4. `mdbook build` ve `mdbook test` (varsa) ile kontrol edin.
5. Değişikliklerinizi commit edin (`git commit -m 'feat: yeni konu eklendi'`).
6. Dalınızı gönderin (`git push origin feature/yeni-konu`).
7. Bir **Pull Request** açın.

---

## 👤 Yazar & İletişim (Author & Contact)

- **Gökay BURUÇ**
  - GitHub: [@gokayburuc](https://github.com/gokayburuc)

---

## 📄 Lisans (License)

Bu proje [MIT Lisansı](LICENSE) kapsamında lisanslanmıştır. Detaylar için repo içeriğini inceleyebilirsiniz.
