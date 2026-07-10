# Nmap Tarama, Çıktı ve Performans Seçenekleri Rehberi (Cheat Sheet)

Bu rehber, siber güvenlik dünyasının en popüler ağ tarama aracı olan **Nmap** (Network Mapper) platformunun temel ve gelişmiş parametrelerini, mantığıyla birlikte Türkçe olarak açıklamaktadır. GitHub deponuzda bir başvuru kılavuzu olarak kullanabilirsiniz.

---

## 1. Tarama Seçenekleri (Scanning Options)

Bu parametreler Nmap'in hedefi nasıl keşfedeceğini, hangi portlara nasıl yaklaşacağını ve ağ üzerinde nasıl davranacağını belirler.

| Nmap Seçeneği | Açıklama | Mantığı ve Amacı |
| :--- | :--- | :--- |
| **`10.10.10.0/24`** | Hedef ağ aralığı (Target network range). | Taramanın yapılacağı IP bloğunu belirtir (Örn: 10.10.10.1 ile 10.10.10.254 arasındaki tüm cihazlar). |
| **`-sn`** | Port taramasını devre dışı bırakır (Host Discovery / Ping Scan). | Portları taramaz, sadece ağda hangi cihazların aktif (ayakta) olduğunu tespit etmek için kullanılır. |
| **`-Pn`** | ICMP Echo İsteklerini (Ping) devre dışı bırakır. | Hedeflerin açık olup olmadığını kontrol etmek için ping atmaz. Cihazları doğrudan aktif kabul ederek port taramasına geçer. Güvenlik duvarlarının ping bloklarını aşmak için kritiktir. |
| **`-n`** | DNS Çözümlemesini devre dışı bırakır. | IP adreslerini alan adlarına (Domain name) çevirmeye çalışmaz. Bu durum tarama hızını ciddi oranda artırır. |
| **`-PE`** | ICMP Echo İstekleri ile ping taraması gerçekleştirir. | Hedefin ayakta olup olmadığını anlamak için standart ICMP Echo (Type 8) paketleri gönderir. |
| **`--packet-trace`** | Gönderilen ve alınan tüm paketleri gösterir. | Arka planda dönen trafiği (giden/gelen tüm paket seviyesindeki logları) anlık izlemenizi sağlar. Hata ayıklama (debugging) için harikadır. |
| **`--reason`** | Belirli bir sonucun nedenini görüntüler. | Bir portun neden "open", "closed" veya "filtered" olduğunu belirten teknik sebebi (Örn: gelen RST veya SYN-ACK paketini) ekranda raporlar. |
| **`--disable-arp-ping`** | ARP Ping İsteklerini devre dışı bırakır. | Yerel ağda (LAN) varsayılan olarak yapılan hızlı ARP sorgularını kapatır, standart IP seviyesi ham paketler kullanmaya zorlar. |
| **`--top-ports=<sayı>`** | En sık kullanılan belirli sayıdaki popüler portu tarar. | Nmap'in veri tabanında tanımlı, dünya genelinde en çok kullanılan ilk `N` adet portu (Örn: `--top-ports=100`) tarayarak zaman kazandırır. |
| **`-p-`** | Tüm portları tarar. | 1 ile 65535 arasındaki olası tüm TCP/UDP portlarını eksiksiz tarar. Uzun sürer ama gözden hiçbir şey kaçmaz. |
| **`-p22-110`** | Belirtilen aralıktaki portları tarar. | Sadece 22 ile 110 arasındaki (22 ve 110 dahil) portları tarar. |
| **`-p22,25`** | Sadece belirtilen özel portları tarar. | Sadece hedefteki 22 (SSH) ve 25 (SMTP) portlarını kontrol eder. |
| **`-F`** | En popüler 100 portu tarar (Fast Scan). | Hızlı tarama modudur. Nmap veri tabanındaki en sık kullanılan ilk 100 portu tarayıp hızla sonuç verir. |
| **`-sS`** | TCP SYN Taraması yapar (Stealth / Half-Open Scan). | Tam bir TCP bağlantısı kurmaz (Three-way handshake tamamlanmaz). Port açık ise gelen SYN-ACK'a RST ile yanıt verilir. Hedef sistemde log bırakma ihtimali daha düşüktür. |
| **`-sA`** | TCP ACK Taraması yapar. | Güvenlik duvarı (Firewall) kurallarını haritalandırmak için kullanılır. Portların açık olup olmadığını değil, kuralların filtreli (`filtered`) olup olmadığını anlamaya yarar. |
| **`-sU`** | UDP Taraması gerçekleştirir. | DNS, DHCP, SNMP gibi UDP protokolü kullanan servisleri tespit etmek için ham UDP paketleri gönderir. TCP'ye göre daha yavaştır. |
| **`-sV`** | Keşfedilen servislerin versiyonlarını tarar. | Açık portlarda çalışan yazılımların adını ve tam versiyon numarasını (Örn: Apache httpd 2.4.41) öğrenmek için banner kapma ve prob gönderme işlemleri yapar. |
| **`-sC`** | Varsayılan Nmap betikleriyle (NSE) tarama yapar. | Nmap Scripting Engine (NSE) kütüphanesindeki en güvenli, yaygın ve bilgi toplayıcı varsayılan (`default`) scriptleri çalıştırır. |
| **`--script <betik>`** | Belirtilen özel betikleri kullanarak tarama yapar. | Sizin seçeceğiniz özel NSE betiklerini (Örn: güvenlik zafiyeti tarayan `vuln` kategorisi veya özel bir script) devreye sokar. |
| **`-O`** | İşletim Sistemi (OS) algılaması yapar. | Hedef cihazın TCP/IP yığın (stack) davranışlarını inceleyerek hangi işletim sistemini (Windows, Linux, iOS vs.) kullandığını tahmin eder. |
| **`-A`** | Agresif Tarama Modu. | Tek bir parametreyle İşletim Sistemi Tespiti (`-O`), Servis Versiyon Tespiti (`-sV`), Betik Taraması (`-sC`) ve Traceroute (Rota takibi) işlemlerinin hepsini bir arada yapar. |
| **`-D RND:5`** | Hedefi tararken 5 adet rastgele Sahte Kimlik (Decoy) kullanır. | Kendi IP adresinizi gizlemek veya hedef sistem loglarında kalabalık yaratıp analizini zorlaştırmak için araya 5 tane rastgele sahte IP adresi serpiştirir. |
| **`-e <arayüz>`** | Kullanılacak ağ arayüzünü (Interface) belirtir. | Taramada hangi ağ kartının (Örn: `eth0`, `wlan0`, `tun0`) kullanılacağını manuel olarak seçmenizi sağlar. |
| **`-S 10.10.10.200`** | Tarama için kaynak IP adresini değiştirir (IP Spoofing). | Paketlerin giden kaynağını belirtilen IP adresiymiş gibi gösterir. Yanıtları doğrudan alamazsınız ancak yanıltma amacıyla kullanılır. |
| **`-g <port>`** | Tarama için kaynak port numarasını (Source Port) belirtir. | Nmap'in paketleri gönderirken bilgisayarınızda hangi portu (Örn: DNS için 53, HTTP için 80) maske olarak kullanacağını belirler. Bazı zayıf firewall kurallarını aşmaya yarar. |
| **`--dns-server <ns>`** | DNS çözümlemesini belirtilen isim sunucusu üzerinden yapar. | Sisteminizin varsayılan DNS sunucusu yerine, sizin tanımladığınız özel bir DNS sunucusunu sorgular için kullanmaya zorlar. |

---

## 2. Çıktı Seçenekleri (Output Options)

Nmap tarama sonuçlarının gelecekte analiz edilebilmesi, raporlanabilmesi veya başka araçlara (Metasploit vb.) aktarılabilmesi için farklı formatlarda kaydedilmesini sağlar.

| Nmap Seçeneği | Açıklama | Mantığı ve Amacı |
| :--- | :--- | :--- |
| **`-oA <dosya_adı>`** | Sonuçları mevcut tüm formatlarda kaydeder. | Tarama çıktısını aynı anda Normal (`.nmap`), Grep edilebilir (`.gnmap`) ve XML (`.xml`) formatlarının üçünde birden tek seferde kaydeder. En profesyonel yaklaşımdır. |
| **`-oN <dosya_adı>`** | Sonuçları normal metin formatında kaydeder. | Ekranda gördüğünüz standart Nmap çıktısının birebir aynısını düz bir metin dosyası (`.txt` / `.nmap`) olarak kaydeder. |
| **`-oG <dosya_adı>`** | Sonuçları "grepable" (filtrelenebilir) formatta kaydeder. | Linux terminalindeki `grep`, `awk`, `cut` gibi komutlarla IP ve port bilgilerini kolayca süzebilmeniz için her hostu tek bir satıra yazacak şekilde özel biçimlendirir. |
| **`-oX <dosya_adı>`** | Sonuçları XML formatında kaydeder. | Veriyi yapılandırılmış XML biçiminde tutar. Bu çıktı, raporlama yazılımları veya zafiyet yönetim araçları tarafından kolayca okunup işlenebilir. |

---

## 3. Performans ve Zamanlama Seçenekleri (Performance Options)

Bu parametreler tarama hızını optimize etmek, ağ bandını doğru kullanmak ve IDS/IPS (Saldırı Tespit ve Engelleme Sistemleri) cihazlarına yakalanmamak adına zamanlamaları ayarlamak için kullanılır.

| Nmap Seçeneği | Açıklama | Mantığı ve Amacı |
| :--- | :--- | :--- |
| **`--max-retries <sayı>`** | Belirli portlar için maksimum yeniden deneme sayısını ayarlar. | Paketin kaybolma ihtimaline karşı Nmap'in bir porta en fazla kaç kez tekrar istek göndereceğini sınırlar. Düşük sayı hız kazandırır ancak paket kaybı olan ağlarda doğruluk payını düşürebilir. |
| **`--stats-every=5s`** | Her 5 saniyede bir tarama durumunu gösterir. | Uzun süren taramalarda, işlemin yüzde kaçının tamamlandığını ve kalan tahmini süreyi her 5 saniyede bir ekrana yazdırarak durumu takip etmenizi sağlar. |
| **`-v / -vv`** | Detaylı çıktı (Verbose) modunu etkinleştirir. | Tarama esnasında arka planda bulunan açık portları veya aşamaları anında ekrana basar. `-vv` detayın dozunu daha da artırır. |
| **`--initial-rtt-timeout 50ms`** | Başlangıç RTT (Round-Trip Time) zaman aşımını ayarlar. | İlk gönderilen paketin yanıtı için beklenecek süreyi sınırlar. Çok hızlı yerel ağlarda süreyi düşük tutmak taramayı inanılmaz hızlandırır. |
| **`--max-rtt-timeout 100ms`** | Maksimum RTT zaman aşımını ayarlar. | Nmap'in bir pakete yanıt almak için maksimum ne kadar süre (Örn: 100 milisaniye) bekleyeceğini belirler. Bu süreyi aşan portlar doğrudan düşmüş veya yanıtsız kabul edilir. |
| **`--min-rate 300`** | Saniyede gönderilecek minimum paket sayısını belirler. | Nmap'in hızı yavaşlatmasını engeller ve saniyede en az 300 paket göndermeye zorlar. Büyük ağları çok hızlı taramak için idealdir. |
| **`-T <0-5>`** | Hazır zamanlama şablonunu (Timing Template) belirtir. | Taramayı hızlandırmak veya yavaşlatmak için 0'dan 5'e kadar şablon seçer:<br>• **T0 (Paranoid) / T1 (Sneaky):** IDS'lerden kaçmak için çok yavaştır.<br>• **T2 (Polite):** Bant genişliğini az tüketir.<br>• **T3 (Normal):** Varsayılan mod.<br>• **T4 (Aggressive):** Hızlı ve kararlı ağlar için önerilen mod.<br>• **T5 (Insane):** Aşırı hızlı ve agresif, ağı çökertebilir veya veri kaçırabilir. |
