# Envanter Takip Sistemi: Sunucu Kurulumu

Bu depo yalnızca kurulum paketlerini (**Releases**) barındırır. Uygulama Docker ile tek komutla kurulur ve yerel ağda çalışır: veritabanı (MariaDB), uygulama ve HTTPS sağlayan ters vekil (Caddy) birlikte ayağa kalkar.

## Gerekenler

- Ubuntu 24.04 LTS sunucu (root veya `sudo` yetkisi)
- İlk kurulum sırasında internet (paket ve Docker indirilir). İnternetsiz sunucu için bkz. [İnternetsiz kurulum](#internetsiz-kurulum)
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

Sunucuda Docker kurulu olmalıdır. Docker'ı da internetsiz kurmak gerekiyorsa paket, Docker `.deb` dosyalarıyla ayrıca hazırlanır (geliştiriciye danışın).

## Bilinmesi gerekenler

- **Saat doğru olmalıdır:** iki adımlı doğrulama kodları saate bağlıdır. İnternet yoksa saati elle veya yerel NTP ile eşitleyin; kurulum, saat eşitlenmiyorsa uyarır.
- **Web Push bildirimleri** internet gerektirir; internetsiz ağda çalışmaz. Uygulama içi bildirimler ve e-posta alarmları çalışır.
- **E-posta:** kurumun yerel SMTP sunucusu kullanılabilir. Ayarlar sayfasından girilir.
- **Güvenlik duvarı:** seçtiğiniz portu açın: `sudo ufw allow 8180/tcp`.
- **`.env` dosyasını yedekleyin ve paylaşmayın.** İçindeki `APP_SECRET` değişirse kayıtlı e-posta parolaları ve 2FA anahtarları okunamaz.
- Port veya adresi değiştirmek için `sudo ./envanter.sh install --port 9443 --host "..."` komutunu kurulum klasöründe tekrar çalıştırın; veriler korunur.

## Sorun giderme

| Belirti | Çözüm |
|---|---|
| `İndirilemedi` | Sunucunun internetine ve `github.com` erişimine bakın. |
| Uygulama "sağlıklı" olmadı | `./envanter.sh logs app` ve `./envanter.sh logs db` çıktısına bakın. |
| Tarayıcı bağlanmıyor | Port açık mı (`ufw`), adres doğru mu, `./envanter.sh status` çalışıyor mu kontrol edin. |
| Sertifika uyarısı | Yukarıdaki "Sertifika uyarısı" bölümündeki kök sertifikayı cihaza kurun. |
| Kurulum anahtarı gerekli | `sudo grep SETUP_TOKEN /opt/envanter/canli/.env` |
