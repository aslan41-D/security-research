# UFW (Uncomplicated Firewall) Kurulum ve Yapılandırma Rehberi

---

## 🇹🇷 Türkçe

### 📌 Giriş

Daha önce Lynis ile bir tarama yaptığınızı ve sisteminizde bir firewall (güvenlik duvarı) bulunmadığını varsayarak, bu yazıda adım adım UFW nasıl kurulur ve yapılandırılır, bunu öğreneceğiz.

### UFW Nedir?

**UFW (Uncomplicated Firewall)**, Linux sistemlerde karmaşık ağ kurallarını yönetmeyi kolaylaştıran bir güvenlik duvarı arayüzüdür. Kurumsal firewall sistemlerinden farkı, arka planda çalışan teknolojileri basit komut satırlarına indirgemesidir.

Aslında UFW tek başına bir güvenlik duvarı değildir. Linux çekirdeğinde yer alan **iptables** (veya modern sistemlerde **nftables**) sistemini yöneten bir ön arayüzdür. Kullanım kolaylığı sayesinde hata yapma riskini azaltır.

**Örnek karşılaştırma:**

| Araç | Komut |
| :--- | :--- |
| **iptables** | `sudo iptables -A OUTPUT -p tcp --dport 22 -j ACCEPT` |
| **UFW** | `sudo ufw allow 22` |

Görüldüğü gibi UFW, aynı işlemi çok daha basit bir komutla yapmanızı sağlar.

> **Not:** UFW, yerel bir sunucuyu korumak için kullanılır ancak kurumsal cihazlar gibi **Katman 7 (Application Layer)** incelemesi yapamaz. Yani geçen trafiğin içeriğini analiz edemez; yalnızca gelen verinin belirtilen portuna bakarak yapılandırmanıza göre izin verir veya reddeder.

---

### 1. UFW'yi Kur

```bash
# Arch Linux / Manjaro
sudo pacman -S ufw

# Debian / Ubuntu
sudo apt install ufw

# RHEL / Fedora
sudo dnf install ufw
```

Ardından systemd servisini başlatın ve önyüklemeye ekleyin:

```bash
sudo systemctl enable --now ufw.service
```
Önemli Not: Eğer sisteminizde iptables.service etkinse, UFW ile çakışacaktır. UFW kullanmaya başlamadan önce iptables.service'in etkin olmadığından emin olun. systemctl status iptables komutuyla kontrol edebilirsiniz.


## 2. Politikaları Ayarla (Gelen Reddet, Giden İzin Ver)

Bu, güvenlik duvarı yapılandırmasının en önemli adımıdır. Dışarıdan gelen ve sizin başlatmadığınız tüm bağlantı isteklerini reddederken, sizin başlattığınız giden bağlantılara izin verir.

```bash
# Gelen tüm bağlantıları varsayılan olarak reddet
sudo ufw default deny incoming

# Giden tüm bağlantıları varsayılan olarak izin ver
sudo ufw default allow outgoing
```
## 3. Güvenlik Duvarını Etkinleştir
```bash
sudo ufw enable
```
Bu komut, mevcut SSH bağlantınızı kesebileceğine dair bir uyarı gösterebilir. Eğer fiziksel olarak o makinenin başındaysanız y yazarak devam edebilirsiniz. SSH üzerinden bağlıysanız, önce SSH portuna izin vermeniz gerekir (bir sonraki adımda anlatılıyor).

4. Gerekli Hizmetlere İzin Ver
Örneğin, bu makineye 22 numaralı porttan SSH ile bağlanıyorsanız, izin vermeniz gerekir; aksi takdirde bağlantınız kopacaktır.

```bash
sudo ufw allow ssh
```
Ekstra güvenlik istiyorsanız, olası brute-force saldırılarına karşı SSH bağlantılarını sınırlayabilirsiniz:

```bash
sudo ufw limit ssh
```
Bu komut, aynı IP adresinden belirli bir süre içinde çok fazla bağlantı denemesi yapılmasını otomatik olarak engeller.

## 5. Durumu Kontrol Et
```bash
sudo ufw status verbose
```
Çıktıda şunları görmelisiniz:

```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     LIMIT       Anywhere
22/tcp (v6)                LIMIT       Anywhere (v6)
```
Status: active → Güvenlik duvarı çalışıyor.

Default: deny (incoming), allow (outgoing) → Varsayılan politikalar doğru.

22/tcp LIMIT → SSH için brute-force koruması aktif.

Lynis taramasında "Firewall is not installed" uyarısı aldıysanız, UFW en hızlı ve en güvenli çözümdür. Ancak UFW'nin bir Layer 3/4 firewall olduğunu unutmayın — uygulama katmanında (Layer 7) koruma için ek çözümlere (WAF, IDS/IPS) ihtiyaç duyulur.

Uyarı: Bu içerik yalnızca eğitim ve yetkili güvenlik test amaçlıdır. Güvenlik duvarını yanlış yapılandırmak, kendi sisteminize erişiminizi engelleyebilir; bu nedenle her zaman önce güvenli bir ortamda test edin.


## 🇬🇧 English
# Introduction
Assuming you have previously run a Lynis scan and found that your system does not have a firewall installed, in this article we will walk through how to install and configure UFW step by step.

## What is UFW?
UFW (Uncomplicated Firewall) is a firewall interface that simplifies managing complex network rules on Linux systems. Unlike enterprise firewall solutions, it reduces the underlying technologies into simple command-line instructions.

UFW is not a standalone firewall. It is a front-end that manages the iptables (or nftables on modern systems) system built into the Linux kernel. Its simplicity reduces the risk of configuration errors.

Example comparison:

Tool	    Command
iptables	  => sudo iptables -A OUTPUT -p tcp --dport 22 -j ACCEPT
UFW => 	sudo ufw allow 22

As shown, UFW achieves the same result with a much simpler command.


Note: UFW is designed to protect a local server, but it cannot perform Layer 7 (Application Layer) inspection like enterprise appliances. It cannot analyze the content of passing traffic; it only checks whether incoming data is destined for a specific port and allows or denies it based on your configuration.

## 1. Install UFW
```bash
# Arch Linux / Manjaro
sudo pacman -S ufw

# Debian / Ubuntu
sudo apt install ufw

# RHEL / Fedora
sudo dnf install ufw
```
Then start the systemd service and enable it on boot:

```bash
sudo systemctl enable --now ufw.service
```
Important Note: If iptables.service is enabled on your system, it will conflict with UFW. Before using UFW, make sure iptables.service is not enabled. You can check with systemctl status iptables.


## 2. Set Policies (Deny Incoming, Allow Outgoing)
This is the most important step in firewall configuration. It denies all incoming connection requests that you did not initiate, while allowing outgoing connections that you initiate.

```bash
# Deny all incoming connections by default
sudo ufw default deny incoming

# Allow all outgoing connections by default
sudo ufw default allow outgoing
```

## 3. Enable the Firewall
```bash
sudo ufw enable
```
This command may display a warning that it could disconnect your current SSH session. If you are physically at the machine, type y to continue. If you are connected via SSH, you must first allow the SSH port (covered in the next step).

## 4. Allow Required Services
For example, if you connect to this machine via SSH on port 22, you must allow it; otherwise, your connection will drop.

```bash
sudo ufw allow ssh
```
If you want extra security, you can rate-limit SSH connections to protect against brute-force attacks:

```bash
sudo ufw limit ssh
```
This command automatically blocks an IP address that makes too many connection attempts within a short period.

## 5. Check the Status
```bash
sudo ufw status verbose
```
You should see output like this:

Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     LIMIT       Anywhere
22/tcp (v6)                LIMIT       Anywhere (v6)

Status: active → The firewall is running.

Default: deny (incoming), allow (outgoing) → Default policies are correct.

22/tcp LIMIT → Brute-force protection is active for SSH.

Key Takeaway
If your Lynis scan reported "Firewall is not installed," UFW is the fastest and safest solution. However, keep in mind that UFW is a Layer 3/4 firewall — application-layer (Layer 7) protection requires additional solutions (WAF, IDS/IPS).

References
[UFW Official Documentation](https://help.ubuntu.com/community/UFW)

[Arch Wiki: Uncomplicated Firewall](https://wiki.archlinux.org/title/Uncomplicated_Firewall)

[Lynis - Security Auditing Tool](https://cisofy.com/lynis/)

## Disclaimer: This content is for educational and authorized security testing purposes only. Misconfiguring a firewall can lock you out of your own system — always test in a safe environment first.









