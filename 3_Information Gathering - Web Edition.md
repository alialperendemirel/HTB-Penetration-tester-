# Web Bilgi Toplama Kılavuzu (Web Information Gathering Cheat Sheet)

Bu rehber, bir web uygulamasına veya alan adına (domain) yönelik sızma testlerinde (Penetration Testing) ve siber güvenlik analizlerinde kullanılan **Pasif ve Aktif Bilgi Toplama (Web Reconnaissance)** tekniklerini, komutlarını ve çalışma mantıklarını kapsamlı bir şekilde Türkçe olarak açıklamaktadır.

---

## 1. WHOIS Sorguları

WHOIS, alan adlarının (domain) kime ait olduğunu, tescil (registrar) firmasını, oluşturulma/biti tarihlerini ve name server (DNS) sunucularını sorgulamak için kullanılan dökümantasyon protokolüdür.

*   **Komut:** `whois example.com`
*   **Mantığı:** Hedef alan adının sahiplik bilgilerini ve hangi altyapı sağlayıcılarında barındırıldığını (Name Serverlar üzerinden) öğrenerek pasif istihbarat (OSINT) toplamak.

---

## 2. DNS Sorguları ve Kayıt Tipleri (`dig`)

DNS (Domain Name System), insan yapımı alan adlarını makine dili olan IP adreslerine çevirir. `dig` komutu doğrudan DNS sunucularından spesifik kayıt tiplerini sorgulamamızı sağlar.

| Kayıt Tipi | Açıklama | Mantığı ve Amacı |
| :--- | :--- | :--- |
| **`A`** | IPv4 Adres Kaydı | Alan adının hangi IPv4 adresine yönlendiğini gösterir (`dig example.com A`). |
| **`AAAA`** | IPv6 Adres Kaydı | Alan adının hangi IPv6 adresine yönlendiğini gösterir (`dig example.com AAAA`). |
| **`CNAME`** | Takma Ad (Alias) | Bir alan adının başka bir ana alan adına yönlendirildiğini gösterir (`dig example.com CNAME`). |
| **`MX`** | Mail Server Kaydı | E-postaları işlemekten sorumlu olan mail sunucularını listeler (`dig example.com MX`). |
| **`NS`** | Name Server Kaydı | Alan adının DNS kayıtlarını yöneten yetkili sunucuları gösterir (`dig example.com NS`). |
| **`TXT`** | Metin Kayıtları | SPF, DKIM, DMARC doğrulamaları veya güvenlik/sahiplik doğrulama kodlarını tutar (`dig example.com TXT`). |
| **`SOA`** | Bölge Yetki Başlangıcı | DNS bölgesinin yönetici e-postasını ve seri numarası gibi idari detayları içerir (`dig example.com SOA`). |

---

## 3. Alt Alan Adı Tespiti (Subdomain Enumeration)

Subdomainler (örn: `admin.example.com`, `dev.example.com`), şirketin test ortamlarını, geliştirici panellerini veya unutulmuş eski uygulamaları barındırabileceği için ana saldırı yüzeyini genişletir.

### A. Aktif Kaba Kuvvet (Subdomain Brute-Forcing)
Bir kelime listesi (wordlist) kullanarak var olan sub-domainleri DNS sunucusuna sorarak doğrular.

*   **Komut:** `dnsenum example.com -f subdomains.txt`
*   **Mantığı:** `dnsenum` aracı, kelime listesindeki her bir satırı (örn: `mail`, `test`, `vpn`) alan adının önüne ekleyerek DNS sorgusu atar. Cihaz geçerli bir IP yanıtı verirse subdomain keşfedilmiş olur.

### B. Bölge Transferi (Zone Transfer - AXFR)
DNS sunucusunun tüm veritabanını tek seferde dışarı sızdıran kritik bir yapılandırma hatasıdır.

*   **Komut:** `dig @ns1.example.com example.com axfr`
*   **Mantığı:** Bir ikincil DNS sunucusu gibi davranarak ana DNS sunucusundaki tüm alan adı haritasını (tüm subdomainler ve IP'ler) talep eder. Eğer sunucu erişim kısıtlaması koymadıysa tüm altyapı sızar.

### C. Sertifika Şeffaflığı (Certificate Transparency - Passive)
Web sitelerine alınan SSL/TLS sertifikaları halka açık CT loglarına işlenir.

*   **Komut:** `curl -s "https://crt.sh/?q=%25.example.com&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u`
*   **Mantığı:** `crt.sh` gibi veritabanlarını sorgulayarak hedefe **hiçbir paket göndermeden** (pasif olarak) geçmişte veya günümüzde SSL sertifikası alınmış tüm subdomainleri süzüp listeler.

---

## 4. Sanal Konak Tespiti (Virtual Host Brute-Forcing)

Tek bir IP adresi üzerinde, HTTP `Host` başlıklarına (Header) göre farklı web siteleri sunulabilir (Virtual Hosting).

*   **Komut:** `gobuster vhost -u http://192.0.2.1 -w hostnames.txt`
*   **Mantığı:** IP adresine atılan HTTP isteklerinde `Host:` başlığını kelime listesinden alınan değerlerle (`Host: dev.example.com`) değiştirir. Dönen yanıt boyutundaki veya HTTP durum kodundaki değişime göre gizli sanal sunucuları bulur.

---

## 5. Web Emekleme (Web Crawling & Spidering)

Sayfalar arasındaki bağlantıları (linkleri) otomatik olarak takip ederek sitenin haritasını çıkarma işlemidir.

*   **Robots.txt İncelemesi:** Web sitelerinin kök dizininde bulunan ve arama motoru botlarına "Taramayın" denilen gizli yolları (`Disallow: /admin`) barındıran kritik bir dosyadır.
*   **Scrapy İle Veri Analizi:**
    *   *Scrapy Çalıştırma:* Python tabanlı Scrapy framework'ü ile site taranıp linkler `example_data.json` dosyasına dökülür.
    *   *Veriyi Süzme Komutu:* `jq -r '.[] | select(.file != null) | .file' example_data.json | sort -u`
    *   *Mantığı:* Çekilen karmaşık JSON verisindeki dosya ve link yollarını ayıklayarak sitedeki gizli parametre veya dosya uzantılarını listeler.

---

## 6. Arama Motoru İle Keşif (Google Dorking / OSINT)

Arama motorlarının indeksleme gücünü özel arama operatörleriyle birleştirip hassas verileri ortaya çıkarma işlemidir.

| Operatör | Açıklama | Örnek Kullanım ve Amacı |
| :--- | :--- | :--- |
| **`site:`** | Aramayı sadece belirli bir alan adı ile sınırlar. | `site:example.com "password reset"` (Sadece hedef sitedeki şifre sıfırlama sayfaları) |
| **`inurl:`** | URL içinde geçen belirli anahtar kelimeleri arar. | `inurl:admin login` (URL'inde 'admin' geçen giriş sayfaları) |
| **`filetype:`** | Sadece belirli bir dosya formatındaki içerikleri getirir. | `filetype:pdf "confidential report"` (Açıkta unutulmuş gizli PDF belgeleri) |
| **`intitle:`** | Sayfa başlığında (Title) geçen ifadeleri arar. | `intitle:"index of" /backup` (Dizin listelemesi açık kalmış yedek klasörleri) |
| **`cache:`** | Sayfanın arama motorundaki geçmiş önbellek görüntüsünü açar. | `cache:example.com` (Siteden silinmiş ama önbellekte kalmış içerikler) |
| **`" "` (Tırnak)** | Yazılan kelimeleri tırnak içindeki tam sırasıyla arar. | `"internal error" site:example.com` (Sitenin verdiği dahili hata çıktıları) |

---

## 7. Web Arşivleri (Wayback Machine)

Bir web sitesinin geçmişte nasıl göründüğünü ve hangi dosyalara sahip olduğunu gösteren zaman makinesidir.

| Özellik | Açıklama | Siber Güvenlikteki Amacı |
| :--- | :--- | :--- |
| **Geçmiş Ekran Görüntüleri** | Sitenin eski versiyonlarını inceleme. | Şu an kaldırılmış ama eskiden aktif olan zafiyetli fonksiyonları veya sayfaları keşfetme. |
| **Gizli Dizinler** | Eski dizin ve dosya yapısı. | Güncel sitede linki kaldırılmış ama sunucudan silinmemiş unutulmuş yedek dosyalarını tespit etme. |
| **İçerik Değişiklikleri** | Zaman içindeki içerik değişim takibi. | Yazılım güncellemelerini ve güvenlik önlemlerinin nasıl evrildiğini analiz etme. |
