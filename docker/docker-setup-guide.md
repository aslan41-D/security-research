# Docker Kurulum ve Kullanım Rehberi

---

## 🇹🇷 Türkçe

### Docker Nedir?

Docker, uygulamaları **container (konteyner)** adı verilen izole ortamlarda çalıştıran bir sanallaştırma yazılımıdır. Bir uygulamayı ve tüm bağımlılıklarını tek bir paket haline getirir; böylece aynı paketi farklı sistemlerde sorunsuzca çalıştırabilirsiniz.

**Temel Mantık:**
- Bir uygulamayı **bir kez paketle** (image oluştur)
- **Her yerde çalıştır** (farklı sunucular, farklı işletim sistemleri)
- **İzole et** (uygulama, sistemi kirletmez; sistem de uygulamayı bozmaz)

### Docker vs Sanal Makine (VM)

| Özellik | Docker Container | Sanal Makine |
| :--- | :--- | :--- |
| **Kaynak Tüketimi** | Çok düşük (MB seviyesi) | Yüksek (GB seviyesi) |
| **Başlatma Süresi** | Saniyeler | Dakikalar |
| **İşletim Sistemi** | Host OS çekirdeğini paylaşır | Kendi OS'u vardır |
| **İzolasyon** | Süreç (process) seviyesinde | Donanım seviyesinde |
| **Kullanım Alanı** | Mikroservisler, CI/CD, test | Tam sistem izolasyonu |

### Neden Pentest'te Docker Kullanırız?

- **Vulhub** gibi platformlardaki zafiyetli ortamları saniyeler içinde ayağa kaldırmak için.
- Farklı CVE'leri izole bir ortamda test etmek için.
- Karmaşık kurulum gerektiren uygulamaları (DVWA, WebGoat vb.) hızlıca çalıştırmak için.
- Ana sisteminizi riske atmadan zafiyetli yazılımlarla çalışmak için.

---

### 1. Kurulum (Kali Linux / Debian / Ubuntu)

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable docker --now
docker --version
```

Kullanıcıyı Docker grubuna ekleme (sudo olmadan kullanmak için):

```bash
sudo usermod -aG docker $USER
newgrp docker
```
### 2. Temel Kavramlar
Image (İmaj): Uygulamanın paketlenmiş hali. Salt-okunur (read-only) bir şablondur.

Container (Konteyner): Image'ın çalışan (runtime) hali. Yazılabilir bir katmandır.

Dockerfile: Image oluşturmak için kullanılan yapılandırma dosyası.

Docker Compose: Birden fazla container'ı tek bir YAML dosyasıyla yönetmeyi sağlar.

Registry: Image'ların saklandığı merkezi yer (Docker Hub, GitHub Container Registry vb.)

Docker Swarm: Yüzlerce container'ı yönetmek ve yük dengelemek (load balancing) için kullanılan orkestrasyon aracı.


3. Temel Komutlar
Image Yönetimi
```bash
# Image indir
docker pull alpine
docker pull nginx

# Yerel image'ları listele
docker images

# Image sil
docker image rm IMAGE_ID
```
Container Çalıştırma
```bash
# Basit çalıştırma
docker container run nginx

# Arka planda (detached) çalıştır, isim ver ve port yönlendir
docker container run -d --name selo -p 80:80 nginx

# Interaktif modda çalıştır ve shell'e gir
docker container run -it alpine /bin/sh
```
Port yönlendirme mantığı: -p HOST_PORT:CONTAINER_PORT

-p 80:80 → Host'un 80 portu, container'ın 80 portuna yönlendirilir.

-p 8080:80 → Host'un 8080 portu, container'ın 80 portuna yönlendirilir.

Container Yönetimi
```bash
# Çalışan container'ları listele
docker ps

# Tüm container'ları listele (durdurulmuş olanlar dahil)
docker ps -a

# Çalışan container'a shell ile bağlan
docker container exec -it e313 /bin/bash

# Container loglarını görüntüle (son 2 satır)
docker container logs selo --tail 2

# Container kaynak kullanımını izle
docker container stats selo

# Container'ı başlat / durdur / öldür
docker container start selo
docker container stop selo
docker container kill selo selo2

# Container'ı sil
docker container rm selo
```

Container İçinde Keşif
```bash
# Container'a gir
docker container exec -it e313 /bin/bash

# İçeride dosya ara
find . -iname "*.html"

# Dosya gezintisi
ls -la /usr/share/nginx/html/
```

## 4. Container'ı Image'a Çevirme (Commit)
Çalışan bir container üzerinde değişiklik yapıp bunu yeni bir image olarak kaydedebilirsin:

```bash
# Çalışan container'ı yeni bir image olarak kaydet
docker commit selo selo:latest

# Yeni image'ı kontrol et
docker images

# Yeni image'dan container çalıştır
docker container run -d --name selo-yeni -p 88:80 selo
```
## 5. Dockerfile ile Kendi Image'ını Oluşturma
Dockerfile, bir image'ın nasıl oluşturulacağını tanımlayan metin dosyasıdır.

Örnek proje yapısı:

```bash
mkdir -p docker/apache
cd docker/apache
nano Dockerfile
```
Örnek Dockerfile:

```dockerfile
FROM ubuntu:22.04

RUN apt update && apt install -y apache2

EXPOSE 80

CMD ["apache2ctl", "-D", "FOREGROUND"]
```
Image'ı build et:

```bash
docker image build --file Dockerfile --tag apache .
docker images
docker container run -d --name apache-test -p 8080:80 apache
```
## 6. Docker Compose ile Çoklu Container Yönetimi
Docker Compose, birden fazla container'ı tek bir YAML dosyasıyla yönetmenizi sağlar. Özellikle Vulhub gibi lab ortamları için vazgeçilmezdir.

Örnek docker-compose.yml:

```yaml
version: '3.3'

services:
  dvwa:
    image: sagikazarmark/dvwa
    restart: always
    ports:
      - "8080:80"

  webgoat:
    image: webgoat/webgoat
    restart: always
    ports:
      - "8081:8080"

  nginx:
    image: nginx
    restart: always
    ports:
      - "8082:80"
```

Kullanım:

```bash
# Tüm servisleri arka planda başlat
docker-compose up -d

# Servisleri durdur ve kaldır
docker-compose down

# Belirli bir servisin loglarını gör
docker-compose logs -f dvwa

# Çalışan servisleri listele
docker-compose ps
```
## 7. Vulhub ile Kullanım (Pentest Lab'leri)
Vulhub, Docker Compose kullanarak gerçek CVE'leri izole bir ortamda çalıştırmanızı sağlar.

```bash
# Vulhub reposunu klonla
git clone https://github.com/vulhub/vulhub.git
cd vulhub

# İlgilendiğin CVE'nin klasörüne git
cd httpd/CVE-2021-41773

# Ortamı ayağa kaldır
docker-compose up -d

# Test et, sömür, öğren...

# İşin bitince ortamı kapat
docker-compose down
```
> Avantajı: Zafiyetli bir uygulamayı kurmak için saatler harcamak yerine, saniyeler içinde hazır hale getirip sömürüye odaklanabilirsiniz.

Docker, pentest ve siber güvenlik dünyasında hız ve izolasyon sağlar. Vulhub gibi kaynaklarla birleştirdiğinizde, gerçek CVE'leri saniyeler içinde test edebileceğiniz bir laboratuvara dönüşür. Ancak Docker'ı yanlış yapılandırmak, kendi sisteminizi riske atmanıza yol açar.

### References
[Docker Official Documentation](https://docs.docker.com/)

[Vulhub - Vulnerable Docker Environments](https://github.com/vulhub/vulhub)

[Docker Security Cheat Sheet - OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)

[Docker Hub](https://hub.docker.com/)

> Uyarı: Bu içerik yalnızca eğitim ve yetkili güvenlik test amaçlıdır. Sistemlere yetkisiz erişim yasa dışıdır.


