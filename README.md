<div align="center">

<img src="logo.svg" width="88" alt="Forte ERP" />

# Forte ERP

**Kurulum paketleri ve sunucu rehberi**

Firma bazlı envanter, stok, satış ve proje takibi. Tek komutla Ubuntu sunucunuza kurulur, yerel ağınızda çalışır.

![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04_LTS-E95420?logo=ubuntu&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)
![MariaDB](https://img.shields.io/badge/MariaDB-11.8-003545?logo=mariadb&logoColor=white)
![HTTPS](https://img.shields.io/badge/HTTPS-Caddy-1F88C0?logo=caddy&logoColor=white)

</div>

---

## Hızlı başlangıç

Sunucuda tek komut yeterlidir:

```bash
curl -fsSL https://github.com/nodefaulthq-lab/crmf_release/releases/latest/download/get.sh | sudo bash -s -- --port 8180
```

Bittiğinde ekrandaki adresi tarayıcıda açın, kurulum anahtarıyla ilk yöneticiyi oluşturun. Başka hiçbir şey kurmanız gerekmez.

> [!NOTE]
> Bu depo yalnızca kurulum paketlerini (**Releases**) barındırır. Kurulum, güncelleme, yedekleme ve sunucu güvenliği adımları aşağıdadır.

## İçindekiler

| Bölüm | İçerik |
|---|---|
| [Gerekenler](#gerekenler) | Sunucu ve ağ ön koşulları |
| [Tek komutla kurulum](#tek-komutla-kurulum) | Komutun yaptıkları ve seçenekler |
| [İlk giriş](#i̇lk-giriş) | Yönetici hesabı oluşturma |
| [Sertifika uyarısı](#sertifika-uyarısı-bir-kez-her-cihazda) | HTTPS güveni, cihaza kurulum |
| [Canlı ve test yan yana](#canlı-ve-test-yan-yana) | İki ortam, yedekle veri aktarma |
| [Yedekleme](#yedekleme) | Elle ve otomatik yedek, S3/R2/SFTP |
| [Güncelleme](#güncelleme) | Tek komutla yeni sürüm |
| [Günlük kullanım](#günlük-kullanım) | `envanter.sh` komutları |
| [İnternetsiz kurulum](#i̇nternetsiz-kurulum) | Paketi elle götürme |
| [Sunucu güvenliği](#sunucu-güvenliği-ubuntu-2404-lts) | 11 adımlık sertleştirme rehberi |
| [Bilinmesi gerekenler](#bilinmesi-gerekenler) | Saat, e-posta, `.env` |
| [Sorun giderme](#sorun-giderme) | Sık görülen sorunlar |

## Neler içerir?

| | |
|---|---|
| 🏢 **Firmalar ve envanter** | Donanım ve yazılım kayıtları, garanti ve lisans takibi, Excel içe/dışa aktarma |
| 📦 **Depo stok** | İrsaliyeli giriş, raf yeri, firmaya çıkış, sayım düzeltmesi |
| 💰 **Satış ve projeler** | Teklif, sipariş, tahsilat, tedarik, proje aşamaları |
| 🔔 **Alarmlar ve takvim** | Süre, stok ve vade alarmları; zil, e-posta, anlık bildirim |
| 💾 **Yedekler** | Uygulamadan indir/yükle/geri yükle, otomatik yedek (yerel, S3/R2, SFTP) |
| 🔐 **Güvenlik** | 2FA, şifreli saklama, rol bazlı erişim, denetim kaydı, sertleştirilmiş kapsayıcılar |

---

## Gerekenler

- Ubuntu 24.04 LTS sunucu (root veya `sudo` yetkisi)
- İlk kurulum sırasında internet (paket ve Docker indirilir). İnternetsiz sunucu için bkz. [İnternetsiz kurulum](#i̇nternetsiz-kurulum)
- Boş bir port (örnekte 8180)
- Yaklaşık 2 GB boş disk

## Tek komutla kurulum

```bash
curl -fsSL https://github.com/nodefaulthq-lab/crmf_release/releases/latest/download/get.sh | sudo bash -s -- --port 8180
```

Komut sırayla şunları yapar:

1. Docker yoksa kurar (`docker.io` ve `docker-compose-v2`).
2. En yeni paketi bu depodan indirir (yaklaşık 180 MB).
3. Paketi `/opt/envanter/canli` klasörüne açar.
4. Parolaları ve uygulama anahtarını sunucuda rastgele üretip `.env` dosyasına yazar.
5. Üç servisi başlatır, uygulamanın hazır olmasını bekler.
6. Adresi ve ilk yönetici kurulum anahtarını ekrana yazar.

Bittiğinde ekranda şuna benzer bir çıktı görürsünüz:

```
Hazır. Adres:  https://192.168.1.20:8180
İlk yönetici kurulum anahtarı:  ...
```

### Seçenekler

| Seçenek | Anlamı | Varsayılan |
|---|---|---|
| `--port N` | Uygulamanın açılacağı port | 8443 |
| `--host "ip, ad"` | Sertifikanın geçerli olacağı adresler (IP ve alan adları, virgülle) | sunucunun IP'leri ve adı |
| `--name ad` | Kurulum adı (klasör `/opt/envanter/<ad>`); canlı ve test yan yana için | `canli` |
| `--like ad` | Şifreleme anahtarını başka bir kurulumdan al (test için) | yok |
| `--bind IP` | Yalnızca bu IP üzerinden dinle (örn. sunucunun yerel ağ IP'si) | tüm arayüzler |
| `--backup-dir /yol` | Yedek klasörü (örneğin bağlı bir ağ paylaşımı) | kurulum klasöründeki `backups` |
| `--version v1.5.0` | Belirli bir sürüm | en yeni |

Örnek: `... | sudo bash -s -- --port 8180 --host "192.168.1.20, envanter.firma.local"`

## İlk giriş

1. Tarayıcıdan `https://<sunucu-ip>:8180` adresini açın.
2. Kurulum ekranında ekrandaki **kurulum anahtarını**, şirket adınızı ve ilk yönetici bilgilerini girin.
3. Anahtarı sunucuda sonra yeniden görmek için: `sudo grep SETUP_TOKEN /opt/envanter/canli/.env`

Kurulum anahtarı yalnızca ilk yöneticiyi oluşturmak içindir; yönetici oluşturulduktan sonra işlevsizdir.

## Sertifika uyarısı (bir kez, her cihazda)

İnternet olmadığı için HTTPS sertifikasını Caddy kendisi üretir. Cihazlar ona ilk başta güvenmez ve uyarı gösterir. Uyarıyı kaldırmak için kök sertifikayı cihaza kurun:

1. `https://<sunucu-ip>:8180/ca.crt` adresinden `envanter-ca.crt` dosyasını indirin.
2. **Windows:** dosyaya çift tıklayın, "Sertifika yükle", "Yerel bilgisayar", "Güvenilen Kök Sertifika Yetkilileri".
3. **Android:** Ayarlar, Güvenlik, Sertifika yükle, "CA sertifikası".
4. **iPhone:** dosyayı açıp profili yükleyin, sonra Ayarlar, Genel, Hakkında, Sertifika Güven Ayarları'ndan açın.

Kamera (QR tarama) ve "uygulamayı yükle" özellikleri yalnızca güvenilen HTTPS adresinde çalışır.

## Canlı ve test yan yana

Aynı sunucuda ayrı veritabanı ve portlarla iki kurulum çalıştırabilirsiniz:

```bash
# canlı
curl -fsSL https://github.com/nodefaulthq-lab/crmf_release/releases/latest/download/get.sh | sudo bash -s -- --port 8180
# test
curl -fsSL https://github.com/nodefaulthq-lab/crmf_release/releases/latest/download/get.sh | sudo bash -s -- --port 8181 --name test --like canli
```

- Test sürümünde üstte sarı "TEST ORTAMI" şeridi görünür.
- `--like canli` sayesinde canlının yedeği testte sorunsuz açılır (e-posta parolası, 2FA gibi şifreli bilgiler dahil).
- **Teste veri girmeye gerek yok:** canlıda *Ayarlar > Yedekler* sayfasından yedek indirin, testte aynı sayfada *Yedek yükle* ile yükleyip geri yükleyin.
- Yeni sürümü önce testte deneyin, sorun yoksa canlıyı güncelleyin.
- Testin otomatik yedek hedefini canlıyla aynı yapmayın.

## Yedekleme

Yedekler uygulamadan yönetilir: **Ayarlar > Yedekler**.

- **Şimdi yedek al:** tüm veri tek dosya olur; listeden indirilir veya geri yüklenir.
- **Otomatik yedek:** her gün seçilen saatte alınır, seçilen gün kadar saklanır.
- **Kopya hedefi:** yalnızca sunucu klasörü; ya da S3 uyumlu depo (AWS S3, Cloudflare R2, MinIO), ya da SFTP sunucusu (IP/adres, kullanıcı, şifre, klasör). "Bağlantıyı dene" düğmesi hedefi kontrol eder.
- **Ağ paylaşımı (SMB/NFS):** paylaşımı sunucuya bağlayın ve `--backup-dir` ile o klasörü gösterin.
- **Geri yükleme** tüm veriyi yedeğin içeriğiyle değiştirir; öncesinde otomatik bir "geri yükleme öncesi" yedek alınır. Yedek ayarları geri yüklemeden etkilenmez.

Yedek klasörü sunucuda `/opt/envanter/<ad>/backups` (veya `--backup-dir` ile verdiğiniz yer). Dosyaları ayrıca başka bir diske veya makineye kopyalamanız önerilir.

## Güncelleme

Kurulumdaki komutu aynen yeniden çalıştırın:

```bash
curl -fsSL https://github.com/nodefaulthq-lab/crmf_release/releases/latest/download/get.sh | sudo bash -s -- --port 8180
```

Kurulu sürüm bulunursa güncelleme yapılır: önce yedek alınır, yeni imajlar yüklenir, uygulama yenilenir. Veriler ve `.env` korunur; veritabanı değişiklikleri açılışta kendiliğinden uygulanır. Test kurulumu için komuta `--name test` ekleyin.

## Günlük kullanım

Kurulum klasöründe (`cd /opt/envanter/canli`):

| Komut | Ne yapar |
|---|---|
| `./envanter.sh status` | Servislerin durumu ve adres |
| `./envanter.sh logs [app\|db\|proxy]` | Kayıtları izler |
| `./envanter.sh stop` / `start` | Durdurur / başlatır (sunucu açılışında kendiliğinden başlar) |
| `./envanter.sh backup` | Komut satırından yedek alır |
| `./envanter.sh restore backups/<dosya>.sql.gz` | Komut satırı yedeğini geri yükler |
| `./envanter.sh ca` | Kök sertifikayı klasöre kopyalar |

## İnternetsiz kurulum

İnternetsiz sunucuda paketi başka bir makineden götürürsünüz:

1. İnternetli bir makinede bu depodaki **Releases** sayfasından `envanter.tar.gz` dosyasını indirin.
2. Dosyayı sunucuya kopyalayın (USB, `scp`).
3. Sunucuda:
   ```bash
   mkdir -p /opt/envanter/canli && sudo tar -xzf envanter.tar.gz -C /opt/envanter/canli --strip-components=1
   cd /opt/envanter/canli && sudo ./envanter.sh install --port 8180
   ```

> [!IMPORTANT]
> Sunucuda Docker kurulu olmalıdır.

Docker'ı da internetsiz kurmak gerekiyorsa paket, Docker `.deb` dosyalarıyla ayrıca hazırlanır (geliştiriciye danışın).

## Sunucu güvenliği (Ubuntu 24.04 LTS)

Uygulama kapsayıcıları zaten sertleştirilmiştir (yetkileri kısıtlı, uygulama dosya sistemi salt okunur, günlükler sınırlı). Aşağıdaki adımlar sunucunun kendisi içindir. Sırayla uygulayın; SSH ile ilgili adımlarda **mevcut oturumu kapatmadan** ikinci bir oturumdan giriş denemesi yapın.

### 1. Güncellemeler ve otomatik güvenlik yamaları

```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```

Güvenlik yamaları kendiliğinden uygulanır. Çekirdek güncellemesi sonrası yeniden başlatma gerekir; gece otomatik yeniden başlatmak için `/etc/apt/apt.conf.d/50unattended-upgrades` dosyasında şunları açın:

```
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "04:00";
```

Uygulama `restart: unless-stopped` ile yeniden açılışta kendiliğinden başlar.

### 2. Yönetici kullanıcı ve SSH

Root ile doğrudan giriş yerine kişisel bir yönetici kullanıcı ve anahtarlı giriş kullanın:

```bash
sudo adduser yonetici && sudo usermod -aG sudo yonetici
```

Kendi bilgisayarınızdan anahtarı gönderin:

```bash
ssh-copy-id yonetici@<sunucu-ip>
```

Anahtarla girişi doğruladıktan sonra parola ile girişi ve root girişini kapatın:

```bash
sudo tee /etc/ssh/sshd_config.d/99-sertlestirme.conf >/dev/null <<'CONF'
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
LoginGraceTime 30
X11Forwarding no
AllowUsers yonetici
CONF
sudo sshd -t && sudo systemctl reload ssh
```

Ayrı bir terminalden `ssh yonetici@<sunucu-ip>` ile girişi doğrulayın, sonra eski oturumu kapatın.

### 3. Güvenlik duvarı (ufw)

Önce SSH'ı, sonra uygulama portlarını yalnızca yerel ağınıza açın (örnekte ağ `192.168.1.0/24`):

```bash
sudo apt install -y ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 192.168.1.0/24 to any port 22 proto tcp
sudo ufw allow from 192.168.1.0/24 to any port 8180 proto tcp
sudo ufw allow from 192.168.1.0/24 to any port 8181 proto tcp
sudo ufw enable
sudo ufw status verbose
```

> [!CAUTION]
> Docker, yayınladığı portları `ufw` kurallarını atlayarak açar. `ufw` tek başına yeterli değildir; aşağıdaki iki önlemden en az birini uygulayın.

**a) Uygulamayı yalnızca sunucunun yerel IP'sine bağlayın** (kolay yol). Kurulumda `--bind 192.168.1.20` verin ya da kurulu sistemde `/opt/envanter/canli/.env` dosyasına `BIND_ADDR=192.168.1.20` satırı ekleyip uygulayın:

```bash
cd /opt/envanter/canli && ./envanter.sh start
```

**b) `DOCKER-USER` zincirine yerel ağ dışını engelleyen kural ekleyin** (örnekte arayüz `eth0`, ağ `192.168.1.0/24`, port 8180):

```bash
sudo apt install -y iptables-persistent
sudo iptables -I DOCKER-USER -i eth0 ! -s 192.168.1.0/24 -p tcp -m conntrack --ctorigdstport 8180 --ctdir ORIGINAL -j DROP
sudo netfilter-persistent save
```

Her yayınlanan port (8181 vb.) için kuralı tekrarlayın. Arayüz adını `ip -br a` ile öğrenin.

### 4. Kaba kuvvet koruması (fail2ban)

```bash
sudo apt install -y fail2ban
sudo tee /etc/fail2ban/jail.d/sshd.local >/dev/null <<'CONF'
[sshd]
enabled = true
maxretry = 4
findtime = 10m
bantime = 1h
CONF
sudo systemctl enable --now fail2ban
sudo fail2ban-client status sshd
```

Uygulamanın kendisi de giriş denemelerini sınırlar ve denetim kaydı tutar.

### 5. Saat eşitleme

İki adımlı doğrulama ve oturum süreleri doğru saate bağlıdır:

```bash
sudo apt install -y chrony
sudo systemctl enable --now chrony
chronyc tracking
```

İnternetsiz ağda `/etc/chrony/chrony.conf` içine yerel bir NTP sunucusu (`server 192.168.1.1 iburst`) ekleyin.

### 6. Dosya izinleri ve veri koruma

```bash
sudo chmod 700 /opt/envanter
sudo chmod 600 /opt/envanter/*/.env
sudo chmod 750 /opt/envanter/*/backups
```

- `.env` dosyasında veritabanı parolaları ve uygulama anahtarı vardır; yedeğini güvenli bir yerde (şifreli) tutun, kimseyle paylaşmayın.
- **Yedek dosyaları tüm verinizi içerir.** Başka makineye kopyalarken şifreli kanal (SFTP, S3 TLS) kullanın; çıkarılabilir diske koyuyorsanız diski şifreleyin.
- Sunucu diskini kurulum sırasında **LUKS** ile şifrelemek (Ubuntu kurulumunda "Disk şifreleme") sunucunun çalınması durumunda veriyi korur.
- Kök sertifikanın özel anahtarı `caddy_data` Docker biriminde durur; sunucuya yetkisiz erişim vermeyin. Sertifika otoritesini yenilemek gerekirse kurulumu silmeden `docker volume rm envanter-canli_caddy_data` ile birimi silip uygulamayı yeniden başlatın; yeni otorite üretilir ve cihazlara yeni kök sertifika kurulur.

### 7. Docker

- Kullanıcıları `docker` grubuna eklemeyin; `docker` grubu pratikte root yetkisidir. Yönetimi `sudo` ile yapın.
- `/etc/docker/daemon.json` dosyasına şunu yazıp `sudo systemctl restart docker` çalıştırın (Docker yeniden başlayınca kapsayıcılar kesintisiz kalır ve yeni kapsayıcılar ek yetki alamaz):

  ```json
  { "live-restore": true, "no-new-privileges": true }
  ```

- Gereksiz imajları temizleyin: `docker image prune -f` (çalışan kurulumlara dokunmaz).

### 8. Sistem ve çekirdek ayarları

```bash
sudo tee /etc/sysctl.d/99-sertlestirme.conf >/dev/null <<'CONF'
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_redirects = 0
kernel.kptr_restrict = 2
kernel.dmesg_restrict = 1
CONF
sudo sysctl --system
```

Kullanılmayan servisleri kapatın: `systemctl list-unit-files --state=enabled` ile bakın, gerekmeyenleri `sudo systemctl disable --now <servis>` ile durdurun. Sunucuda yalnızca SSH ve Docker dinlemelidir.

### 9. Kayıtlar ve izleme

```bash
sudo mkdir -p /var/log/journal && sudo systemctl restart systemd-journald
sudo apt install -y auditd
```

- Kalıcı `journald` kayıtları yeniden başlatmadan sonra da incelenebilir.
- Uygulama kayıtları: `cd /opt/envanter/canli && ./envanter.sh logs app`. Uygulamadaki denetim kaydı (kim, ne zaman, ne yaptı) veritabanında tutulur.
- Disk dolmasını izleyin (`df -h`); yedek klasörü büyüyebilir. Docker günlükleri 10 MB x 3 dosya ile sınırlıdır.

### 10. Uygulama içi güvenlik

- İlk yöneticiden sonra tüm yöneticiler için **iki adımlı doğrulamayı** açın (Hesabım).
- Yalnızca izleyecek kullanıcılara **salt okunur** rol verin; kullanıcıları görevlerine göre sınırlı firmalara bağlayın.
- İlk yönetici oluşturulduktan sonra kurulum anahtarı (`SETUP_TOKEN`) işlevsizdir; değeri paylaşılmışsa bile kötüye kullanılamaz.
- Ayarlar > Yedekler'de **otomatik yedeği** açın ve kopyayı sunucu dışında bir yere (SFTP/S3) gönderin. Geri yüklemeyi düzenli aralıklarla **test ortamında deneyin**.

### 11. Denetim listesi

Kurulumdan sonra kontrol edin:

```bash
sudo ss -tlnp
sudo ufw status verbose
sudo sshd -T | grep -E "permitrootlogin|passwordauthentication"
sudo fail2ban-client status sshd
timedatectl | grep -E "synchronized|NTP"
sudo docker ps --format '{{.Names}} {{.Ports}}'
```

- `ss -tlnp` çıktısında yalnızca SSH ve uygulama portları (`docker-proxy`) görünmelidir.
- Başka bir bilgisayardan `nmap -p 1-10000 <sunucu-ip>` ile yalnızca beklenen portların açık olduğunu doğrulayın.
- İsteğe bağlı kapsamlı tarama: `sudo apt install -y lynis && sudo lynis audit system`. Çıktıdaki önerileri değerlendirin.

## Bilinmesi gerekenler

> [!WARNING]
> **`.env` dosyasını yedekleyin ve paylaşmayın.** İçindeki `APP_SECRET` değişirse kayıtlı e-posta parolaları ve 2FA anahtarları okunamaz.

- **Saat doğru olmalıdır:** iki adımlı doğrulama kodları saate bağlıdır. İnternet yoksa saati elle veya yerel NTP ile eşitleyin; kurulum, saat eşitlenmiyorsa uyarır.
- **Web Push bildirimleri** internet gerektirir; internetsiz ağda çalışmaz. Uygulama içi bildirimler ve e-posta alarmları çalışır.
- **E-posta:** kurumun yerel SMTP sunucusu kullanılabilir. Ayarlar sayfasından girilir.
- **Güvenlik duvarı:** seçtiğiniz portu açın: `sudo ufw allow 8180/tcp`.
- Port veya adresi değiştirmek için `sudo ./envanter.sh install --port 9443 --host "..."` komutunu kurulum klasöründe tekrar çalıştırın; veriler korunur.

## Sorun giderme

| Belirti | Çözüm |
|---|---|
| `İndirilemedi` | Sunucunun internetine ve `github.com` erişimine bakın. |
| Uygulama "sağlıklı" olmadı | `./envanter.sh logs app` ve `./envanter.sh logs db` çıktısına bakın. |
| Tarayıcı bağlanmıyor | Port açık mı (`ufw`), adres doğru mu, `./envanter.sh status` çalışıyor mu kontrol edin. |
| Sertifika uyarısı | Yukarıdaki "Sertifika uyarısı" bölümündeki kök sertifikayı cihaza kurun. |
| Kurulum anahtarı gerekli | `sudo grep SETUP_TOKEN /opt/envanter/canli/.env` |
