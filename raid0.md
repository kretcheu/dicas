# Instalando Debian raid0 UEFI

1- boot live-cd
2- criar partições
3- criar raid0
4- montar
5- instalar debian
6- chroot
7- instalar pacotes
8- preparar grub
9- boot

## 1- boot live-cd

Usei o live-cd-standard sem interface gráfica.

https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/
debian-live-11.2.0-amd64-standard.iso

"Instalar" pacotes:
```
apt update
apt install bash-completion openssh-server gdisk dosfstools debootstrap arch-install-scripts
```

Loguei via ssh
```
ssh user@ip
sudo -s
```

## 2- criar partições

- Criar uma partição de 50MB para UEFI
- Criar demais partições (fiz uma só do restante do disco)

```
sgdisk -z /dev/vda

sgdisk -n 1:0:+50M -t 1:ef00 -c 1:"EFI System" /dev/vda
sgdisk -n 2:0:0 -t 2:fd00 -c 2:"Linux RAID" /dev/vda

sgdisk -z /dev/vdb
sgdisk /dev/vda -R /dev/vdb -G
```


```
fdisk -l

Disk /dev/vda: 4 GiB, 4294967296 bytes, 8388608 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: BE2AC36E-1960-4BE7-8D07-09191E4CE193

Device      Start     End Sectors Size Type
/dev/vda1    2048  104447  102400  50M EFI System
/dev/vda2  104448 8388574 8284127   4G Linux RAID

Disk /dev/vdb: 4 GiB, 4294967296 bytes, 8388608 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: F9D141C3-B4A5-46CF-A591-08B8658EDBDE

Device      Start     End Sectors Size Type
/dev/vdb1    2048  104447  102400  50M EFI System
/dev/vdb2  104448 8388574 8284127   4G Linux RAID

```
Preparar partição EFI
```
mkfs.fat -F 32 /dev/vda1

mount /dev/vda1 /mnt/
mkdir /mnt/EFI
umount /mnt
```

## 3- criar raid
```
mdadm --create /dev/md0 --level=0 --raid-disks=2 /dev/vd[ab]2
```

Criando as partições no raid0

```
sgdisk -z /dev/md0
sgdisk -N 1 -t 1:8300 -c 1:"Linux filesystem" /dev/md0
```

## 4- montar

Criar sistema de arquivos
```
mkfs.ext4 /dev/md0p1

mount /dev/md0p1 /mnt
mkdir -p /mnt/boot/efi

mount /dev/vda1 /mnt/boot/efi
```

## 5- instalar debian
```
debootstrap bullseye /mnt http://deb.debian.org/debian

```
## 6- chroot
```
arch-chroot /mnt

```
## 7- instalar pacotes
```
apt install bash-completion locales
```

Configurando locales:
Modo interativo:
```
dpkg-reconfigure locales
 selecione pt_BR.UTF-8 UTF-8
```

Modo não interativo:
```
sed -i -e 's/# pt_BR.UTF-8 UTF-8/pt_BR.UTF-8 UTF-8/' /etc/locale.gen && \
    echo 'LANG="pt_BR.UTF-8 UTF-8"'>/etc/default/locale && \
    dpkg-reconfigure --frontend=noninteractive locales && \
    update-locale LANG=pt_BR.UTF-8 UTF-8
```

Instalando pacotes: kernel, grub, mdadm
```
apt install linux-image-amd64 grub-efi-amd64 mdadm --no-install-suggests --no-install-recomends
```

Usuários e senhas:
```
passwd
adduser kretcheu
```

```
## 8- preparar grub para o boot
```
grub-install /dev/md0
update-grub
```
Saindo do chroot
```
genfstab -U /mnt > /mnt/etc/fstab
```

## 9- boot
```
umount /mnt/boot/efi
umount /mnt

shutdown -h now
```
