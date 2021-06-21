# Criando um GNUzinho...

1. Requisitos
2. kernel
3. busybox
4. imagem de disco

## 1. Requisitos

- Pacotes de desenvolvimento
- Fontes kernel
- Fontes busybox

## 2. Kernel

1. Preparar .config
2. Compilar

## 3. Busybox

1. Preparar .config
2. Compilar

## 4. Initram

1. Preparar diretório filesystem
2. Copiar busybox
3. Gerar arquivo initram

## 5. Imagem

1. Criar imagem
2. Instalar bootloader
3. Copiar kernel
4. Copiar initram

## 6. Teste

1. Copiar para servidor
2. Boot
3. Testar

## 7. Aprimoramento...

- 2.1/2.2 - Preparar kernel
- 3.1/3.2 - Preparar Busybox
- 4.2 Copiar Busybox
- 4.3 Gerar initram
- 5.3 Copiar kernel
- 5.4 Copiar initram
- 6.1 Copiar para o servidor
- 6.2 Boot
- 6.3 Testar

## 1. Requisitos

### Pacotes necessários

```
apt install git flex bison libncurses-dev build-essential bc kmod cpio liblz4-tool lz4 libncurses-dev libelf-dev libssl-dev syslinux dosfstools
```

### Criando um diretório de trabalho

```
mkdir ~/gnuzinho
cd ~/gnuzinho
```

### Baixando fontes do kernel Linux

- Linux do kernel.org

```
git clone --depth=1 https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git

```

- Linux-Libre

http://linux-libre.fsfla.org/pub/linux-libre/releases

```
wget http://linux-libre.fsfla.org/pub/linux-libre/releases/LATEST-5.12.N/linux-libre-5.12.12-gnu.tar.bz2
tar -xvf linux-libre-5.12.12-gnu.tar.bz2

ln -s linux-5.12.12 linux
```

### Baixando fontes do Busybox

```
cd ~/gnuzinho

wget https://busybox.net/downloads/busybox-1.33.1.tar.bz2
tar xvf busybox-1.33.1.tar.bz2

ln -s busybox-1.33.1 busybox
```

## 2. Preparando Kernel

### Gerando um .config mínimo

```
cd ~/gnuzinho/linux
make tinyconfig
```

### 2.1 Ajustando o .config

```
make menuconfig
```

- General setup ---> Initial RAM filesystem and RAM disk (initramfs/initrd) support ---> yes
  (somente gzip)
- General setup ---> Configure standard kernel features ---> Enable support for printk ---> yes

- 64-bit kernel ---> yes

- Executable file formats / Emulations ---> Kernel support for ELF binaries ---> yes
- Executable file formats / Emulations ---> Kernel support for scripts starting with #! ---> yes

- Device Drivers ---> Generic Driver Options ---> Maintain a devtmpfs filesystem to mount at /dev ---> yes
- Device Drivers ---> Generic Driver Options ---> Automount devtmpfs at /dev, after the kernel mounted the rootfs ---> yes
- Device Drivers ---> Character devices ---> Enable TTY ---> yes
- Device Drivers ---> Character devices ---> Serial drivers ---> 8250/16550 and compatible serial support ---> yes
- Device Drivers ---> Character devices ---> Serial drivers ---> Console on 8250/16550 and compatible serial port ---> yes

- File systems ---> Pseudo filesystems ---> /proc file system support ---> yes
- File systems ---> Pseudo filesystems ---> sysfs file system support ---> yes

------------

- Network Supporting ---> TCP/IP networking

- Device Drivers ---> Network device support --> Ethernet driver support -> Intel 82586..
- Device Drivers ---> Network device support --> Ethernet driver support -> Intel devicesPRO/1000 Gigabit e PRO/1000 PCI-Express

- Device Drivers ---> PCI support ->>

- General setup ---> Configure standard kernel features (expert users) -> Posix Clocks & Timers

- Networking support --> Networking options --> TCP/IP networking -> IP: kernel level autoconfiguration

### 2.2 Compilando kernel

```
make clean
make bzImage
```

### Copiando a imagem do kernel para diretório de trabalho

```
cp arch/x86_4/boot/bzImage ../
```


## 3. Busybox

### Preparando o .config

Zerando...

```
cd ~/gnuzinho/busybox
make allnoconfig
```

### Incluindo as ferramentas no config

```
make menuconfig
```

Exemplos de feramentas mínimas a incluir

- Settings > Support files > 2GB
- Settings > Build static binary (no shared libs)
- Coreutils > cat, du, echo, ls, sleep, uname (change Operating system name to anything you want)
- Console Utilities > clear
- Editors > vi
- Init Utilities > poweroff, reboot, init, Support reading an inittab file
- Linux System Utilities > mount, umount, lspci
- Miscellaneous Utilities > less
- Network Utilities > ip, ping
- Process Utilities > free, ps
- Shells > ash, Optimize for size instead of speed, Alias support, Help builtin

### Compilando....

```
make clean

make
make install
```


## 4. Initram

### 4.1 Preparar diretório filesystem

```
cd ~/gnuzinho
mkdir filesystem
cd filesystem

mkdir -pv {dev,proc,etc/init.d,sys,tmp}
mknod dev/console c 5 1
mknod dev/null c 1 3
```

### Alguns arquivos de configuração

```
cat >> welcome << EOF
Bem vindo ao GNUzinho!
  ____ _   _ _   _     _       _
 / ___| \ | | | | |___(_)_ __ | |__   ___
| |  _|  \| | | | |_  / | '_ \| '_ \ / _ \
| |_| | |\  | |_| |/ /| | | | | | | | (_) |
 \____|_| \_|\___//___|_|_| |_|_| |_|\___/

EOF
```

Inittab
```
cat >> etc/inittab << EOF
::sysinit:/etc/init.d/rc
::askfirst:/bin/sh
::restart:/sbin/init
::ctrlaltdel:/sbin/reboot
::shutdown:/bin/umount -a -r
EOF
```

Init script

```
cat >> etc/init.d/rc << EOF
#!/bin/sh
mount -t proc none /proc
mount -t sysfs none /sys
clear
cat welcome
/bin/sh
EOF
```

### Ajustes de permissões

```
chmod +x etc/init.d/rc
chown -R root:root .
```

## 4.2 Copiando Busybox para filesystem

```
cd ~/gnuzinho/busybox
cp -r _install/* ../filesystem/
```

## 4.3 Gerar arquivo initram

```
cd ~/gnuzinho/filesystem
find . | cpio -H newc -o | gzip -9 > ../rootfs.cpio.gz
```

## 5. Preparando imagem de disco

## 5.1 Criando Imagem de disco

```
cd ~/gnuzinho
dd if=/dev/zero of=gnuzinho.img bs=1k count=2048
```

### Criando FS FAT
```
mkdosfs gnuzinho.img
```
## 5.2 Instalando bootloader

```
syslinux --install gnuzinho.img
```

Criando arquivo de configuração do syslinux

```
cd ~/gnuzinho

cat >> syslinux.cfg << EOF
DEFAULT linux
LABEL linux
  SAY [ BOOTING GNUZinho VERSION 0.1.0 ]
  KERNEL bzImage
  APPEND initrd=rootfs.cpio.gz
EOF

chmod +x syslinux.cfg
```

## Completando imagem de disco

## 5.3 e 5.4
```
mount -o loop gnuzinho.img /mnt
cp bzImage rootfs.cpio.gz syslinux.cfg /mnt/

ls /mnt/
bzImage  ldlinux.c32  ldlinux.sys  rootfs.cpio.gz  syslinux.cfg

umount /mnt
```

## 6. Testes

### Copiar para servidor

```
scp gnuzinho.img kretcheu@servidor:~/vms
```

### Boot

## 7. Aprimorando...

- 2.1/2.2 - Preparar kernel
- 3.1/3.2 - Preparar Busybox
- 4.2 Copiar Busybox
- 4.3 Gerar initram
- 5.3 Copiar kernel
- 5.4 Copiar initram

### Alterando kernel 2.1 / 2.2

```
cd ~/gnuzinho/linux

make menuconfig
make clean
make bzImage

cp arch/x86_4/boot/bzImage ../
```

### Alterando Busybox 3.1/3/2

```
cd ~/gnuzinho/busybox

make menuconfig
make clean
make
make install
```

### Copiando Busybox 4.2
```
cp -r _install/* ../filesystem/
```

### Gerando initram 4.3

```
cd ~/gnuzinho/filesystem
find . | cpio -H newc -o | gzip -9 > ../rootfs.cpio.gz
```

### Copiando kernel e initram 5.3/5.4

```
cd ~/gnuzinhho
mount -o loop gnuzinho.img /mnt
cp bzImage rootfs.cpio.gz /mnt/
umount /mnt
```

### Enviar para o servidor

```
scp gnuzinho.img kretcheu@servidor:~/vms
```

GOTO **Boot**

-----------

### Scripts do Blau

00-requisitos
```
#!/usr/bin/env bash

# Pacotes

pkgs='
git
flex
bison
libncurses-dev
syslinux
build-essential
kmod
cpio
liblz4-tool
lz4
libncurses-dev
libelf-dev
libssl-dev
dosfstools
bc
'

apt install $pkgs
```

01-kernel_org
```
#!/usr/bin/env bash

git clone --depth=1 https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git

```

02-kernel_fsfla
```
#!/usr/bin/env bash

wget http://linux-libre.fsfla.org/pub/linux-libre/releases/LATEST-5.12.N/linux-libre-5.12.12-gnu.tar.bz2
tar -xvf linux-libre-5.12.12-gnu.tar.bz2
ln -s linux-5.12.12 linux
```

03-busybox
```
#!/usr/bin/env bash

wget https://busybox.net/downloads/busybox-1.33.1.tar.bz2
tar xvf busybox-1.33.1.tar.bz2

ln -s busybox-1.33.1 busybox
```

-------
# Material de apoio

- [ramfs-rootfs-initramfs](https://www.kernel.org/doc/Documentation/filesystems/ramfs-rootfs-initramfs.txt)



