# Altyapı ve Servis Bazlı Bilgi Toplama Rehberi (Enumeration Cheat Sheet)

Bu rehber, sızma testlerinde (Penetration Testing) hedef sistemler hakkında bilgi toplamak, açık servisleri, paylaşımları ve kullanıcı adlarını süzmek için kullanılan temel ve gelişmiş komutları mantığıyla birlikte Türkçe olarak açıklamaktadır.

---

## 1. Altyapı Bazlı Bilgi Toplama (Infrastructure-based Enumeration)

Hedefin doğrudan sunucularına dokunmadan önce, dış kaynaklardan (OSINT) altyapı hakkında istihbarat toplama aşamasıdır.

| Komut | Açıklama | Mantığı ve Amacı |
| :--- | :--- | :--- |
| **`curl -s https://crt.sh/\?q\=<target-domain>\&output\=json \| jq .`** | Sertifika Şeffaflığı (Certificate Transparency) kontrolü. | Şirketlerin web siteleri için aldığı SSL/TLS sertifikaları halka açık loglarda tutulur. Bu komut, hedef domaine ait geçmişte veya şu an oluşturulmuş tüm alt alan adlarını (subdomain) pasif olarak keşfetmeyi sağlar. |
| **`for i in $(cat ip-addresses.txt);do shodan host $i;done`** | Listede bulunan her bir IP adresini Shodan üzerinde tarar. | Elinizdeki IP listesini döngüye sokarak, Shodan API'si üzerinden bu cihazların internete açık portlarını, zafiyetlerini ve servis bilgilerini hedefe tek bir paket bile göndermeden (pasif olarak) öğrenmenizi sağlar. |

---

## 2. Servis Bazlı Bilgi Toplama (Host-based Enumeration)

Hedef makineler doğrudan tespit edildikten sonra, üzerlerinde çalışan spesifik servislerin (portların) derinlemesine incelenmesi aşamasıdır.

### FTP (File Transfer Protocol - Port 21)
Dosya paylaşımı amacıyla kullanılan bu protokolde en büyük hedef, yetkisiz erişimler veya eski versiyon zafiyetleridir.

| Komut | Açıklama | Mantığı ve Amacı |
| :--- | :--- | :--- |
| **`ftp <FQDN/IP>`** | Hedefteki FTP servisi ile etkileşime girer. | Standart FTP istemcisini başlatarak hedef sunucuya kullanıcı adı ve şifre ile bağlanmayı (veya `anonymous` girişini denemeyi) sağlar. |
| **`nc -nv <FQDN/IP> 21`** | Netcat ile FTP servisine bağlanır. | Netcat kullanarak porta ham (raw) bağlantı kurar. Amacı, FTP servisinin tam adını ve sürümünü (Banner) ekrana yazdırarak versiyon zafiyeti aramaktır. |
| **`telnet <FQDN/IP> 21`** | Telnet ile FTP servisine bağlanır. | Tıpkı Netcat gibi çalışır; porta TCP bağlantısı açarak sunucunun karşılama mesajını (Banner) okumanızı ve komut göndermenizi sağlar. |
| **`openssl s_client -connect <FQDN/IP>:21 -starttls ftp`** | Şifreli FTP servisine güvenli bağlantı kurar. | FTP bağlantısı FTPS (SSL/TLS) ile şifrelenmişse, düz metin araçları çalışmaz. Bu komut şifreleme katmanını (handshake) aşarak servis ile konuşmanızı sağlar. |
| **`wget -m --no-passive ftp://anonymous:anonymous@<target>`** | FTP sunucusundaki tüm dosyaları indirir. | Eğer sunucuda "Anonymous" (isimsiz/herkese açık) giriş aktifse, sunucudaki tüm dosyaları aynalayarak (`mirror`) kendi bilgisayarınıza indirir ve analiz etmenizi sağlar. |

### SMB (Server Message Block - Port 445)
Özellikle Windows ağlarında dosya, yazıcı ve IPC (prosesler arası iletişim) paylaşımlarını yöneten, sızma testlerinin en sevdiği protokoldür.

| Komut | Açıklama | Mantığı ve Amacı |
| :--- | :--- | :--- |
| **`smbclient -N -L //<FQDN/IP>`** | SMB üzerinde Null Session (Boş oturum) kimlik doğrulaması yapar. | `-N` (Şifresiz) ve `-L` (Listele) parametreleri ile kullanıcı adı ve şifre girmeden, sunucunun dışarıya hangi paylaşımları (klasörleri) sunduğunu görmeye çalışır. |
| **`smbclient //<FQDN/IP>/<share>`** | Belirli bir SMB paylaşımına bağlanır. | Keşfedilen spesifik bir klasörün (Örn: `/Backups`) içine girerek dosya okuma/yazma yetkilerini test etmek için kullanılır. |
| **`rpcclient -U "" <FQDN/IP>`** | RPC kullanarak hedefle etkileşime girer. | Boş bir kullanıcı adı (`-U ""`) ile Uzaktan Prosedür Çağrısı (RPC) oturumu açar. Bağlantı başarılı olursa domain kullanıcıları, gruplar ve sistem bilgileri sorgulanabilir. |
| **`samrdump.py <FQDN/IP>`** | Impacket betiği ile kullanıcı adı listelemesi yapar. | Security Account Manager (SAM) veri tabanını uzaktan sorgulayarak, hedef sistemde kayıtlı olan yerel kullanıcı hesaplarını ortaya çıkarmaya çalışır. |
| **`smbmap -H <FQDN/IP>`** | SMB paylaşımlarını listeler ve yetkileri kontrol eder. | Hedefteki paylaşılan klasörleri hızlıca tarar ve sizin o klasörlerde okuma (`READ`) veya yazma (`WRITE`) yetkinizin olup olmadığını tablo halinde sunar. |
| **`crackmapexec smb <FQDN/IP> --shares -u '' -p ''`** | Null session kullanarak paylaşımları listeler. | Güçlü bir otomasyon aracı olan CrackMapExec ile boş kullanıcı ve şifre kullanarak sistemdeki tüm paylaşılan klasörleri tarar. |
| **`enum4linux-ng.py <FQDN/IP> -A`** | enum4linux ile gelişmiş SMB analizi yapar. | Hem Windows hem Linux (Samba) sistemlerden kullanıcılar, gruplar, paylaşımlar, şifre politikaları ve işletim sistemi bilgilerini tek komutla agresif şekilde toplar (`-A`). |

### NFS (Network File System - Port 2049)
Linux dünyasının ortak dosya paylaşım protokolüdür. Yanlış yapılandırmalar (Örn: herkese açık yetki verilmesi) doğrudan veri sızıntısına yol açar.

| Komut | Açıklama | Mantığı ve Amacı |
| :--- | :--- | :--- |
| **`showmount -e <FQDN/IP>`** | Kullanılabilir NFS paylaşımlarını gösterir. | Hedef Linux sunucunun dışarıya hangi klasörleri ihraç ettiğini (export) ve bu klasörlere hangi IP adreslerinin erişebileceğini listeler. |
| **`mount -t nfs <FQDN/IP>:/<share> ./target-NFS/ -o nolock`** | Belirli bir NFS paylaşımını yerel klasöre bağlar (Mount). | Hedefteki uzak klasörü, kendi Linux makinenizdeki `./target-NFS` dizinine bağlar. Böylece hedef dosya sistemine kendi bilgisayarınızdaymış gibi erişebilirsiniz. |
| **`umount ./target-NFS`** | Bağlanmış NFS dosya sistemini ayırır (Unmount). | İşiniz bittiğinde, oluşturduğunuz dosya sistemi bağlantısını güvenli bir şekilde sonlandırır. |

### DNS (Domain Name System - Port 53)
IP adreslerini isimlere çeviren sistemdir. Buradan elde edilecek alt alan adları, saldırı yüzeyini genişletir.

| Komut | Açıklama | Mantığı ve Amacı |
| :--- | :--- | :--- |
| **`dig ns <domain.tld> @<nameserver>`** | Belirtilen DNS sunucusuna NS isteği atar. | Hedef domainin ad sunucularının (Name Server - DNS yönetiminden sorumlu sunucular) hangileri olduğunu öğrenmek için kullanılır. |
| **`dig any <domain.tld> @<nameserver>`** | DNS sunucusuna ANY (Tüm kayıtlar) isteği atar. | Hedef domaine ait MX, TXT, A, CNAME gibi tüm DNS kayıtlarını tek seferde çekmeye çalışır. |
| **`dig axfr <domain.tld> @<nameserver>`** | DNS Bölge Transferi (Zone Transfer) isteği atar. | **En kritik DNS zafiyetidir.** Eğer DNS sunucusu yanlış yapılandırılmışsa, o domaine ait tüm alt alan adları ve IP haritası (tüm DNS veritabanı) tek bir komutla saldırgana sızdırılır. |
| **`dnsenum --dnsserver ...`** | DNS üzerinde kaba kuvvet (Brute Force) ile subdomain arar. | Bölge transferi (AXFR) başarısız olursa, elindeki kelime listesini (`subdomains.list`) kullanarak `test.hedef.com`, `admin.hedef.com` gibi alt alan adlarının var olup olmadığını tek tek deneyerek bulur. |

### SMTP (Simple Mail Transfer Protocol - Port 25)
E-posta gönderme protokolüdür. Eski sürümlerde kullanıcı doğrulaması yapılarak sistemdeki geçerli e-posta adresleri sızdırılabilir.

| Komut | Açıklama | Mantığı ve Amacı |
| :--- | :--- | :--- |
| **`telnet <FQDN/IP> 25`** | Telnet ile SMTP (E-posta) servisine bağlanır. | SMTP portuna bağlanarak `VRFY` (Kullanıcı doğrula) veya `EXPN` (Listeyi genişlet) komutlarıyla içeride hangi kullanıcı hesaplarının veya e-posta adreslerinin aktif olduğunu anlamak için manuel köprü kurar. |

### IMAP/POP3 (Port 143, 993, 110, 995)
E-posta okuma protokolleridir. Genellikle ele geçirilen kullanıcı bilgilerinin doğruluğunu test etmek için kullanılırlar.

| Komut | Açıklama | Mantığı ve Amacı |
| :--- | :--- | :--- |
| **`curl -k 'imaps://...' --user <user>:<pass>`** | cURL ile şifreli IMAP servisine giriş yapar. | Güvenli IMAP (IMAPS) portu üzerinden eldeki kullanıcı adı ve şifrenin geçerli olup olmadığını test eder. Giriş başarılıysa e-posta kutusunun özetini çeker. |
| **`openssl s_client -connect <FQDN/IP>:imaps`** | IMAPS (Şifreli IMAP) servisine SSL/TLS tüneli açar. | 993 portundaki şifreli trafiği aşarak, arka plandaki IMAP komut satırı ile doğrudan konuşabilmenizi sağlar. |
| **`openssl s_client -connect <FQDN/IP>:pop3s`** | POP3s (Şifreli POP3) servisine SSL/TLS tüneli açar. | 995 portundaki şifreli POP3 servisine bağlanarak, mail istemcisi gibi davranıp düz metin komutlarla e-postaları sorgulamanıza izin verir. |

### SNMP (Simple Network Management Protocol - Port 161/162 UDP)
Ağ cihazlarını (Router, Switch, Sunucu) izlemek ve yönetmek için kullanılır. Eğer gizli kelime (Community String) tahmin edilebilirse sistemdeki her şey sızar.

| Komut | Açıklama | Mantığı ve Amacı |
| :--- | :--- | :--- |
| **`snmpwalk -v2c -c <string> <FQDN/IP>`** | Belirlenen topluluk dizisi (Community String) ile OID'leri sorgular. | Eğer topluluk ismi (Örn: `public` veya `private`) doğruysa, cihazın ram kullanımı, çalışan süreçleri, ağ kartları ve hatta bazen şifreleri gibi tüm sistem ağacını (MIB) aşağıya doğru listeler. |
| **`onesixtyone -c community-strings.list <FQDN/IP>`** | SNMP topluluk isimlerini kaba kuvvetle (Brute Force) tahmin eder. | Cihaza bağlanabilmek için gereken gizli kelimeyi (`public`, `internal`, `cisco` vb.) bir kelime listesi yardımıyla çok hızlı bir şekilde tahmin etmeye çalışır. |
| **`braa <community string>@<FQDN/IP>:.1.*`** | SNMP nesne kimliklerini (OID) toplu ve hızlıca sorgular. | `snmpwalk` aracına göre çok daha agresif ve hızlıdır; tek bir istek dalgasıyla ASN.1 veri ağacındaki tüm kritik sistem bilgilerini havuz gibi çeker. |

### Veri Tabanları (MySQL - 3306 / MSSQL - 1433)
Verilerin kalbidir. Kimlik bilgileri elde edildikten sonra içerideki hassas verileri çekmek için bağlanılır.

| Komut | Açıklama | Mantığı ve Amacı |
| :--- | :--- | :--- |
| **`mysql -u <user> -p<password> -h <FQDN/IP>`** | MySQL sunucusuna uzaktan bağlanır. | Elde edilen veya varsayılan (Örn: `root`) kimlik bilgileri ile MySQL veri tabanı yönetim paneline komut satırından erişim sağlar. |
| **`mssqlclient.py <user>@... -windows-auth`** | Windows kimlik doğrulaması ile MSSQL'e bağlanır. | Impacket aracı vasıtasıyla, hedef Microsoft SQL sunucusuna Windows domain haklarını (NTLM/Kerberos) kullanarak bağlanmayı dener. Sızma testlerinde hak yükseltme (Privilege Escalation) için sıkça kullanılır. |

### IPMI (Intelligent Platform Management Interface - Port 623 UDP)
Sunucuların ana kartına doğrudan bağlı olan, bilgisayar kapalıyken bile yönetim sağlayan süper yetkili bir servistir.

| Komut | Açıklama | Mantığı ve Amacı |
| :--- | :--- | :--- |
| **`msf6 auxiliary(scanner/ipmi/ipmi_version)`** | Metasploit ile IPMI versiyon tespiti yapar. | Hedef yönetim kartının IPMI 1.5 mi yoksa 2.0 mı kullandığını anlar. Bu bilgi, kullanılacak zafiyet türünü seçmek için kritik önem taşır. |
| **`msf6 auxiliary(scanner/ipmi/ipmi_dumphashes)`** | IPMI kullanıcı şifre özetlerini (Hash) dışarı aktarır. | **IPMI 2.0 protokol zafiyetini sömürür.** Sunucu, şifre doğrulaması yapmadan önce kullanıcı şifresinin HMAC-SHA1 özetini (hash) saldırgana gönderir. Bu hash'ler daha sonra bilgisayarınızda kırılarak (Crack) şifre ele geçirilir. |

### Linux Uzaktan Yönetim (SSH - Port 22)
Linux sistemlerin yönetim kapısıdır. Güvenli kabul edilir ancak yapılandırma hataları incelenmelidir.

| Komut | Açıklama | Mantığı ve Amacı |
| :--- | :--- | :--- |
| **`ssh-audit.py <FQDN/IP>`** | SSH servisine karşı güvenlik denetimi yapar. | SSH sunucusunun desteklediği şifreleme algoritmalarını (KexAlgorithms, Ciphers) inceler. Eski veya zayıf şifreleme yöntemleri (Örn: 3DES, MD5) varsa bunları raporlar. |
| **`ssh <user>@<FQDN/IP>`** | Standart SSH istemcisi ile giriş yapar. | Kullanıcı adı ve şifre kombinasyonu ile Linux terminaline uzaktan erişim sağlamak için kullanılır. |
| **`ssh -i private.key <user>@<FQDN/IP>`** | Özel anahtar (Private Key) kullanarak SSH girişi yapar. | Şifre yerine, sızma testinde bir şekilde ele geçirilmiş olan bir RSA/ED25519 özel anahtar dosyasını kimlik kartı gibi göstererek şifresiz oturum açar. |
| **`ssh <user>@... -o PreferredAuthentications=password`** | Sunucuyu şifre tabanlı giriş yapmaya zorlar. | Sunucu varsayılan olarak sadece anahtar (`publickey`) kabul ediyor olabilir. Bu parametre ile istemciyi sadece klasik şifre sormaya zorlayarak, şifre deneme (Brute force) saldırılarının önünü açarız. |

### Windows Uzaktan Yönetim (RDP - 3389 / WinRM - 5985, 5986 / WMI - 135)
Windows sistemlerin uzaktan yönetim ve komut çalıştırma servisleridir.

| Komut | Açıklama | Mantığı ve Amacı |
| :--- | :--- | :--- |
| **`rdp-sec-check.pl <FQDN/IP>`** | RDP servisinin güvenlik ayarlarını kontrol eder. | Uzak Masaüstü (RDP) bağlantısının NLA (Network Level Authentication) destekleyip desteklemediğini veya "Man-in-the-Middle" (Ortadaki Adam) saldırılarına karşı korumasız olup olmadığını analiz eder. |
| **`xfreerdp /u:<user> /p:"<pass>" /v:<FQDN/IP>`** | Linux üzerinden RDP (Görsel Masaüstü) bağlantısı açar. | Elinizdeki geçerli Windows kullanıcı bilgileri ile hedef makinenin grafiksel ekranına (GUI) doğrudan bağlanmanızı sağlar. |
| **`evil-winrm -i <FQDN/IP> -u <user> -p <password>`** | WinRM servisi üzerinden PowerShell oturumu açar. | Windows Uzaktan Yönetim (WinRM) portu açıksa ve admin haklarına sahipseniz, sisteme RDP gibi ağır bir grafik ekran yükü bindirmeden doğrudan hızlı bir PowerShell terminali açarak sızmanızı sağlar. |
| **`wmiexec.py <user>:"<pass>"@... "<command>"`** | WMI servisi üzerinden uzaktan komut çalıştırır. | Windows Management Instrumentation (WMI) kullanarak hedef sistemde iz bırakmadan tek satırlık bir komut (Örn: `whoami`) çalıştırır ve sonucunu size döner. Antivirüslere yakalanma ihtimali düşüktür. |

### Oracle TNS (Port 1521)
Büyük kurumsal şirketlerin kullandığı Oracle veri tabanı yönetim servisidir. Yapısı karmaşık olduğu için özel araçlar gerektirir.

| Komut | Açıklama | Mantığı ve Amacı |
| :--- | :--- | :--- |
| **`./odat.py all -s <FQDN/IP>`** | ODAT aracıyla Oracle üzerinde tam tarama yapar. | Oracle Database Attacking Tool (ODAT) kullanarak; SID (Veri tabanı kimliği) tahmini, şifre denemesi ve versiyon açıklarını tek bir hamlede otomatik olarak tarar. |
| **`sqlplus <user>/<pass>@<FQDN/IP>/<db>`** | Standart Oracle istemcisi ile veri tabanına bağlanır. | Geçerli Oracle kimlik bilgileriyle doğrudan SQL komut satırına erişerek veri tabanı tablolarını sorgulamanızı sağlar. |
| **`./odat.py utlfile -s ... --putFile ...`** | Oracle RDBMS sistemi aracılığıyla sunucuya dosya yükler
