# 🧱 Atlas Sunucusu Kurulum Rehberi

Fiziksel sunucu: **64 GB RAM, 8 TB HDD**
İşletim sistemi: **Ubuntu Server 25.04 (Live Server ISO)**
Amaç: **Sanal makineler üzerinde web sunucuları çalıştırmak**

---

## 📦 1. Donanım ve Hazırlık

- Sunucuya klavye, mouse ve monitör bağla.
- BIOS'a gir:
  - UEFI aktif olmalı.
  - Virtualization (Intel VT-x / AMD-V) açık olmalı.


## 🚀 2. Ubuntu Server Kurulumu

- USB'den boot et.
- Kurulum sırasında:
  - Disk yapılandırmasını **LVM** ile yap.
  - `openssh-server` kurulumunu seç.
  - Güncellemeleri kur.

Kurulum sonrası terminale geç.

---

## 🔧 3. Temel Sistem Yapılandırması

```bash
sudo apt update && sudo apt upgrade -y
sudo hostnamectl set-hostname atlas-host
sudo apt install net-tools htop curl git unzip ufw fail2ban -y
```

---

## 🧠 4. KVM ile Sanallaştırma Kurulumu

### A. KVM + libvirt + virt-manager kurulumu
```bash
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager -y
```

### B. Kullanıcıyı libvirt grubuna ekle
```bash
sudo usermod -aG libvirt $(whoami)
```

### C. KVM desteğini doğrula
```bash
kvm-ok
```

---

## 🌐 5. Ağ (Bridge) Yapılandırması

`/etc/netplan/01-netcfg.yaml` dosyasına örnek bridge config:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp3s0:
      dhcp4: no
  bridges:
    br0:
      interfaces: [enp3s0]
      dhcp4: yes
```

```bash
sudo netplan apply
```

---

## 💻 6. Sanal Makine Kurulumu (CLI Örneği)

ISO'yu `/var/lib/libvirt/boot` içine koy:
```bash
sudo mkdir -p /var/lib/libvirt/boot
sudo cp ubuntu-25.04.iso /var/lib/libvirt/boot/
```

Sanal makine oluştur:
```bash
sudo virt-install \
  --name webserver1 \
  --vcpus 4 \
  --memory 8192 \
  --cdrom /var/lib/libvirt/boot/ubuntu-25.04.iso \
  --disk size=50 \
  --os-type linux \
  --network bridge=br0 \
  --graphics none
```

---

## 🌍 7. Web Sunucusu Kurulumu (VM içinde)

Sanal makineye SSH ile bağlan:
```bash
ssh username@ip_adresi
```

Gerekli bileşenleri kur:
```bash
sudo apt install nginx nodejs postgresql -y
```

---

## 🔐 8. Güvenlik

### UFW Kurulumu:
```bash
sudo ufw allow ssh
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

### Fail2Ban:
```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

---

## 📈 9. İzleme ve Performans

```bash
sudo apt install glances vnstat -y
```

Glances:
```bash
glances
```

---

## 🧯 10. Ekstra Notlar

- UPS kullanımı önerilir.
- RAID yapılandırması gerekiyorsa kurulum öncesi yapılandırılmalı.
- Snapshot/backup için `rsync`, `restic`, `borg` gibi çözümler incelenmeli.
- Sunucu BIOS/Firmware güncellemeleri yapılmalı.

---

> Geliştirici: **Atlas Yazılım**  
> Rehber: "Fiziksel Sunucudan Web Servise"

