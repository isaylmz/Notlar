<details>
	<summary>İçindekiler</summary>

+ [Kullanıcı Hesap Yönetimi](#Kullanıcı-Hesap-Yönetimi)
	+ [Sudo Komutunun Kullanımı](#Sudo-Komutunun-Kullanımı)
	+ [Güçlü Parola Yönetimi](#Güçlü-Parola-Yönetimi)
	+ [Yeni Hesap Açma](#Yeni-Hesap-Açma)
	+ [Hesap Geçerlilik Süresinin Yönetimi](#Hesap-Geçerlilik-Süresinin-Yönetimi)
	+ [Kilitli Hesapların Yönetilmesi](#Kilitli-Hesapların-Yönetilmesi)

+ [Dosya Yönetimi](#Dosya-Yönetimi)
	+ [Dosya Sahiplikleri](#Dosya-Sahiplikleri)
	+ [Suid, Guid ve Sticky Bit Kavramları](#Suid,-Guid-ve-Sticky-Bit-Kavramları)

+ [Şifreleme İşlemleri](#Şifreleme-İşlemleri)
	+ [ecryptfs Sisteminin Kullanımı](#ecryptfs-Sisteminin-Kullanımı)
	+ [GPG Uygulamaları](#GPG-Uygulamaları)

+ [Erişim Denetimi](#Erişim-Denetimi)
	+ [iptables Kullanımı](#iptables-Kullanımı)
	+ [ssh Servis Kurulumu](#ssh-Servis-Kurulumu)
	+ [ssh Parolasız Erişim Yönetimi](#ssh-Parolasız-Erişim-Yönetimi)

</details>

## Kullanıcı Hesap Yönetimi

### Sudo Komutunun Kullanımı

Kim olduğunu görmek:
```bash
whoami
```

id olarak tanımını görmek:
```bash
id
```

Üyesi olduğumuz grupları görmek:
```bash
groups
```

Sudo komutu kullanıldığında hangi yetkilere sahip olunduğunu görmek:
```bash
sudo -l
```

**Not**: İlk sudo kullanıldığında şifre sorar ve 5 dakika sürer.

Sudo yetkisini kapatmak:
```bash
sudo -k
```

Doğrudan root hesabına geçiş yapmak:
```bash
sudo -i
```

Root hesabından çıkış yapmak:
```bash
exit
```

Yeni kullanıcı eklemek:
```bash
sudo useradd user_name
```

Yeni kullanıcıya şifre belirlemek:
```bash
sudo passwd user_name
```

Kullanıcı değiştirmek:
```bash
su user_name
```

Kullanıcıların isimleri, id değerleri, yetkilendirmeleri ve ev dizinlerini yer alır:
```bash
cat /etc/passwd
```

**Not**: Sıradan herhangi bir kullanıcı bu dosyayı (passwd) görebilir.

Kullanıcıyı sudo grubuna almak:
```bash
sudo usermod -aG sudo user_name
```

Kullanıcı şifrelerinin hesh değerleri yer alır:
```bash
sudo cat /etc/shadow
```

Sudo yetkisini kimlerin hangi niteliklerle kullanabildiği yer alır:
```bash
sudo cat /etc/sudoers
```

sudoers dosyasını düzenlemek için kullanılan komut:
```bash
sudo visudo
```

sudoers dosyasına yeni bir satır ekleyerek, sudo yetkilerini kullanıcı için düzenlemek:
```
user_name   ALL=(ALL)   ALL
```

sudoers dosyasına yeni bir satır ekleyerek, sudo yetkilerini kullanıcı için düzenlemek:
```
user_name   ALL=(ALL)   /bin/systemctl status networking

user_name   ALL=(ALL)   /sbin/fdisk
```

 İçerisine aşağıdaki komutların eklendiği bir .sh dosyası çalıştığında root olma yetkisinin olmamasına rağmen root kullanıcısı olmuş oluruz:
```sh
#!/bin/bash

sudo -i
```

Sadece more komutunu kullanabilsin ve sadece ssh_config dosyasını düzenleyebilsin:
```
user_name   ALL=(ALL)   /bin/more /etc/ssh/ssh_config
```

**NOT**: ssh_config dosyasına `!bash` satırı eklendiğinde root kullanıcısa geçilmiş olur.

### Güçlü Parola Yönetimi

Yardımcı paketi indirmek:
```bash
sudo apt install libpam-pwquality

#Parola politikasını yönetmek için kullanılacak yardımcı paket.
```

Yapılandırma dosyasını incelemek:
```bash
sudo gedit /etc/security/pwquality.conf

# Dosya içerisinde bulunan satırları, "#" silerek etkinleştirebilirsiniz.
```

### Yeni Hesap Açma

Yeni bir kullanıcı, kullanıcı için ev dizini ve varsayılan kabuk tanımlamak:
```bash
sudo user add user_name -m -d /home/user_name -s /bin/bash
```

Kullanıcı için parola belirlemek:
```bash
sudo passwd user_name
```

**NOT**: Bir kullanıcı diğer kullanıcıya ait dizine girebilir.

Başka kullanıcıların girmesini önlemek için:
```bash
sudo chmod 700 *
```

Ev dizinlerinin varsayılan yetkilendirmellerini düzenlemek:
```bash
sudo nano /etc/login.defs

# UMASK değerini 077'ye çevirdiğinizde.
# Artık yeni oluşturulan kullanıcı kilitlenmiş ve başka kullanıcıların erişimine kapatılmış olur.
```

### Hesap Geçerlilik Süresinin Yönetimi

Geçerlilik süresi tanımlayarak yeni kullanıcı oluşturmak:
```bash
sudo useradd user_name -m -d /home/user_name -s /bin/bash -e 2026-06-30
```

Geçerlilik süresini değiştirmek:
```bash
sudo usermod user_name -e 2025-06-30
```

Kullanıcıya hesaba ilk girişinde parola belirletmek:
```bash
# Önce kullanıcıya ait parola belirlenir.
sudo paswd user_name

# Daha sonra parolanın son kullanım süresini değiştirilir.
sudo passwd -e user_name 
```

### Kilitli Hesapların Yönetilmesi

Düzenleme yapılacak yapılandırma dosyasını açmak:
```bash
sudo gedit /etc/pam.d/common-auth
```
Aşağıdaki satır eklenir
```
# here are the per-package modules (the "Primary" block)

auth   required   pam_tally2.so onerr=faile deny=3 unlock_time=600 even_deny_root audit
```

**NOT**: 
+ auth   required : Oturum açmak için yetkilendirme.
+ pam_tally2.so : Eklenen kütüphane
+ onerr=faile : Hata durumunda tepki ver.
+ deny=3 : 3 denemeden sonra bunu sağla.
+ unlock_time=600 : 10 dakika boyunca kilitli durum dursun.
+ even_deny_root : root olsa bile bu hesabı kilitle.

Kullanıcıların yapmış olduğu oturum denemelerini görmek:
```bash
sudo pam_tall2
```

Kitli hesabı tekrar açmak:
```bash
sudo usermod -U user_name
```

Hesabı kilitlemek:
```bash
sudo passwd -l user_name
```

Hesabı açmak:
```bash
sudo passwd -u user_name
```

## Dosya Yönetimi

### Dosya Sahiplikleri

Dosya sahipliğini görmek:
```bash
echo "yeni dosya" > file_name.txt
ls -l

# Çıktı:
# -rw-rw-r-- 1 isaylmz isaylmz   11 Haz 23 17:59  file.txt
```

**NOT**:
+ rw : Sahip olan kişi okuma ve yazma yetkisi
+ r : Üye olduğu grup okuma yetkisi
+ r : Diğerleri okuma yetkisi

Sahipliği değiştirmek:
```bash
sudo chown user_name file_name.txt
```

Grubunu değiştirmek:
```bash
sudo chown :group_name file_name.txt
```

İkisini birden değiştirmek:
```bash
sudo chown user_name:group_name file_name.txt
```

Yeni grup oluşturmak:
```bash
sudo groupadd group_name
```

Grubun varlığını anlamak:
```bash
sudo tail /etc/group
```

Klosördeki dosyaların tamamını belli bir grubun sahipliğine üye yapmak:
```bash
sudo chown :group_name -R folder_name/
```

Kullanıcıya çalıştırma yetkisi vermek:
```bash
chmod u+x file.sh
```

Gruba çalıştırma yetkisi vermek:
```bash
chmod g+x file.sh
```

Diğerlerine çalıştırma yetkisi vermek:
```bash
chmod o+x file.sh
```

**Not**: Sayısal Karşılıkları
+ r = 4
+ w = 2
+ x = 1
Eğer istersek sayısal değerlerinin toplamı olarak da değerler atayabiliriz.

Bütün yetkileri sadece kullanıcıya vermek:
```bash
chmod 700 file.sh

# 700 = ugo

ls -l

# Çıktı:
# -rwx------ 1 isaylmz isaylmz        11 Haz 23 17:59  file.txt
```

Yetkileri sayısal olarak görmek:
```bash
stat -c %a file.sh
```

Klasör içerisindeki bütün dosyaları ve sayısal olarak yetkilendirmelerini görmek:
```bash
stat -c "%n %a" *
```

### Suid, Guid ve Sticky Bit Kavramları

**Suid Bit**: tanımlanan dosyanın root yetkisine sahip olmasını sağlar.

**NOT**: Suid Bit yetkisine sahip bir nmap interactive olarak çalıştırılıp (`nmap --interactive`) shell ortamı çağrıldığında (`!sh`) root olduğumuzu görebiliriz (`whoami`).

**Sticky Bit**: Oluşturulan bir dosyayı oturum açan başka bir kullanıcı tarafından değiştirilmesi ya da silinmesini engeller.

```bash
ls -ld /tmp

# drwxrwxrwt 21 root root 4096 Haz 26 22:03 /tmp

# t harfi Sticky Bit ile tanımlandığını gösterir
```

## Şifreleme İşlemleri

### ecryptfs Sisteminin Kullanımı

Gerekli kütüphanenin indirilmesi:
```bash
sudo apt install ecryptfs-utils
```

```bash
sudo mkdir /YeniDizin
# Klasörü kendi üzerinden mount ediyoruz.
sudo mount -t ecryptfs /YeniDizin /YeniDizin
# Passphhrase giriyoruz.
# Şifreleme algoritmasını seçiyoruz.
# Key bytes seçiyoruz.
# Enable filename encryption (y/n) = Dosya ismi şifrelensin mi (e/h)

# Dizin içerisinde dosya oluşturuyoruz.
sudo nano çokgizli.txt

# Klasörü umount ediyoruz.
sudo umount /YeniDizin
# Artık dizin içeriği okunamıyor.

# Tekrar aynı parametrelerle klasörü kendi üzerinden mount ediyoruz.
sudo mount -t ecryptfs /YeniDizin /YeniDizin -o key=passphrase,ecryptfs_cipher=aes,ecryptfs_key_bytes=16,ecryptfs_passthrough=no,ecryptfs_enable_filename_ctypto=yes
# Artık dizin içeriği okunabiliyor.
```

### GPG Uygulamaları

**Simetrik Şifreleme**: Tek bir şifre kullanarak şifrelemek ve yine onunla açmak.

**Asimetrik Şifreleme**: Ben özel anahtarımla şifrelerken karşı taraf benim açık anahtarımla şifrelediğim dosyayı açması.

Simetrik şifreleme yapmak:
```bash
# Üzerinde deneme yapak için bir .txt dosyası oluşturuyoruz.
echo "Gizli bir dosya." > gizli.txt

# Dosyaları listeliyoruz. 
ls -l
# -rw-rw-r-- 1 isaylmz isaylmz        17 Haz 27 22:28  gizli.txt
# -rw-rw-r-- 1 isaylmz isaylmz        94 Haz 27 22:30  gizli.txt.gpg

# Dosyayı şifreliyoruz.
gpg -c gizli.txt

# Dosyayı geri döndürülemez bir şekilde siliyoruz.
shred -u -z gizli.txt.gpg

# Dosyayı aynı şifre ile açıyoruz.
sudo gpg -d gizli.txt.gpg
```

Hem açık hem özel anahtar üretmek:
```bash
# Özel anahtarı üretmek
sudo gpg --full-generate-key
```

Açık anahtar ürettiğinden emin olmak:
```bash
cd .gnupg/

# Gerekli dosya ve klasörlerin üretildiğiini görebilirsiniz.
ls
```

Kendine özel açık anahtarı üretmek:
```bash
gpg --export -a -o user_name_public_key.txt
```

Başka bir kullanıcının açık anahtar alt yapısını tanımlamak:
```bash
sudo gpg --import /home/user_name/.gnupg/user_name_public_key.txt
```

Özel anahtarla şifrelemek:
```bash
# Üzerinde deneme yapak için bir .txt dosyası oluşturuyoruz.
echo "Gizli bir dosya." > gizli.txt

# Dosyayı şifreliyoruz.
gpg -s -e gizli.txt
# Alıcı tanımlamamız istenecek. Açık anahtarını kullanmak istediğimiz kullanıcıyı tanımlıyoruz.

# Açık anahtarını kullandığımız kullanıcıya geçiyoruz.
su user_name

# Dosyayı açıyoruz.
sudo gpg -d gizli.txt
```

## Erişim Denetimi

### iptables Kullanımı

Mevcut güvenlik duvarı yapılandırılmasını görmek:
```bash
# Numerik olarak görüntülenmesini sağlar.
sudo iptables -L -n
```

**NOT**: Bu kurallar ipv4 için geçerli sadece eğer ipv6 için de kurallar tanımlamak istiyorsak `ip6tables` kullanılmalı.

Kaynağı benim sunucum olan bir trafik karşı taraf ile bir bağlantı kurduysa trafiğe izin vermek:
```bash
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

ssh servisi yani 22 numaralı portu açmak:
```bash
sudo iptables -A INPUT -p tcp --dport ssh -j ACCEPT
```

Adlandırma/DNS sunucusuyla iletişime geçmesini kabul etmek:
```bash
# DNS protokolü 53 numaralı porttan giden udp ve tcp protokolüdür.
sudo iptables -A INPUT -p tcp --dport 53 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 53 -j ACCEPT
```

127.0.0.1 localhost trafiğini tanımlamak:
```bash
sudo iptables -I INPUT 1 -i lo -j ACCEPT
```

**NOT**: `-A` inputun kategorisinin en altına eklerken `-I` ilk başa ekler.

**NOT**: Eğer güvenli bir sunucu kurmak istiyorsanız ping trafiğini baştan kapatın. Böylelikle hackerlar sizi göremesin.

Sistemin sağlığının kontrolü açısından faydalı olacak ICMP paket türlerine müsade etmek:
```bash
# ICMP tür 3: ualaşılamaz mesajları tanımlar.
sudo iptables -A INPUT -m conntrack -i icmp --icmp-type 3 --ctstate NEW,ESTABLISHED,RELATED -j ACCEPT
# Zaman aşımına uğrayan paketleri tanımlar.
sudo iptables -A INPUT -m conntrack -i icmp --icmp-type 11 --ctstate NEW,ESTABLISHED,RELATED -j ACCEPT
# Bozuk başlık değerine sahip olan ICMP türlerini tanımlar.
sudo iptables -A INPUT -m conntrack -i icmp --icmp-type 12 --ctstate NEW,ESTABLISHED,RELATED -j ACCEPT
```

**NOT**: Eğer ICMP trafik bir sıkıntı nedeniyle ulaşmıyorsa bunu engellememiz iyi bir şey değildir, bizim bunu farketmemiz gerekiyor.

Kabul et kurallarını tanımladıktan sonra bütün her şeyi kes kuralını tanılamak:
```bash
# sudo iptables -A INPUT -j DROP
# Bu şekilde tanımlanması durumunda DROP tan sonra tanımlanan ACCEPT algılanmaz.
# O yüzden bunun yerine temel politika olarak tanımlayabiliriz.

# -P temel politika tanımlamak için kullanılır.
sudo iptables -P INPUT DROP 
```

**NOT**: `DROP` paketi tamamen kes başka bir şey yapma kale bile alma demek.

Tanımanan kuralların uçmamasını sağlamak için kullanılan paket:
```bash
sudo apt install iptables-persistent
```

Kuralları sıfırlamak:
```bash
sudo iptables -P INPUT ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
# -F flash yani tertemiz et
sudo iptables -F
```

### ssh Servis Kurulumu

ssh anahtarı oluşturmak:
```bash
ssh-keygen
```

Açık anahtarı görüntülemek:
```bash
cat /home/user_name/.ssh/id_rsa.pub
```

**NOT**: Açık anahtarı başka bir makinanın `/home/user_name/.ssh` dizinine kaydettikten sonra açık anahtarı kaydettiğimiz makinaya özel anahtarımızla bağlanabiliriz.

Yetkilendirmeden emin olmak:
```bash
# ssh bağlantısı eğer bundan daha düşük bir özel anahtar yetkisine sahip olursa bizi uyaracak ve düzgün çalışmayacaktır.
chmod 600 id_rsa
```

Özel anahtar ile bağlantı kurmak:
```bash
ssh -i /home/user_name/.ssh/id_rsa user_name@ip_adress
```

### ssh Parolasız Erişim Yönetimi

ssh servisi mevcut mu anlamak:
```bash
netstat -antp | grep 22

# ya da
# systemctl status ssh
```

ssh servisini ayağa kaldırmak:
```bash
sudo apt install openssh-server
```

ssh etkinleştirmek:
```bash
sudo systemtl enable ssh
```
