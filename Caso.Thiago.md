# Caso transplante para o mesmo HD
## Situação atual

     fdisk -l


    Disk /dev/sda: 447.1 GiB, 480103981056 bytes, 937703088 sectors
    Disk model: KINGSTON SA400S3
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0xd2eda978

    Device     Boot   Start       End   Sectors   Size Id Type
    /dev/sda1  *       2048   1171455   1169408   571M 83 Linux
    /dev/sda2       1173502 937701375 936527874 446.6G  5 Extended
    /dev/sda5       1173504 937701375 936527872 446.6G 83 Linux

    Disk /dev/mapper/sda5_crypt: 446.6 GiB, 479485493248 bytes, 936495104 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes

    Disk /dev/mapper/maria--vg-raiz: 46.1 GiB, 49480204288 bytes, 96641024 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes

    Disk /dev/mapper/maria--vg-home: 400.5 GiB, 430004240384 bytes, 839852032 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes

    lsblk

    NAME                 MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
    sda                    8:0    0 447.1G  0 disk
    ├─sda1                 8:1    0   571M  0 part  /boot
    ├─sda2                 8:2    0     1K  0 part
    └─sda5                 8:5    0 446.6G  0 part
      └─sda5_crypt       254:0    0 446.6G  0 crypt
        ├─maria--vg-raiz 254:1    0  46.1G  0 lvm   /
        └─maria--vg-home 254:2    0 400.5G  0 lvm   /home

 `df -h`

    Filesystem                  Size  Used Avail Use% Mounted on
    udev                        3.9G     0  3.9G   0% /dev
    tmpfs                       790M   18M  772M   3% /run
    /dev/mapper/maria--vg-raiz   46G   11G   33G  26% /
    tmpfs                       3.9G   67M  3.8G   2% /dev/shm
    tmpfs                       5.0M  4.0K  5.0M   1% /run/lock
    tmpfs                       3.9G     0  3.9G   0% /sys/fs/cgroup
    /dev/mapper/maria--vg-home  395G   32G  343G   9% /home
    /dev/sda1                   563M  114M  420M  22% /boot
    tmpfs                       790M   72K  790M   1% /run/user/1000
    /dev/sdb                    458G   40G  395G  10% /media/andrade/0e0a7f90-7bbb-4251-a073-bf26a2eefcf3

# Fazendo os backups tar

    mkdir /antigo
    mount /dev/mapper/maria--vg-raiz /antigo
    cd /antigo
    tar -cvf /media/andrade/0e0a7....fcf3/barra-antigo.tar *

    mount /dev/mapper/maria--vg-home /antigo/home
    cd /antigo/home
    tar -cvf /media/andrade/0e0a7....fcf3/home-antigo.tar *

    mount /dev/sda1 /antigo/boot
    cd /antigo/boot
    tar -cvf /media/andrade/0e0a7....fcf3/boot-antigo.tar *

## no live-cd - Começando o transplante

    fdisk /dev/sda
    manda um g -> para criar o particionamento gpt

    > **capaz de dar umas reclamadas, mas vai fundo.**
    > dai acho que devemos criar as partições:

    sda1 100M EFI tipo1
    sda2 400M linux (padrão)
    sda3 o resto linux padrão

    > **vc tá ligado no fdisk, vai de boa?**
    > **Acho que seria assim:**

    sda1 - EFI  - 100M
    sda2- /boot - 400M
    sda3- Lucks
     -> LVM
     -> -> mapper/raiz
     ->  -> mapper/home

    root@debian:/media/user# fdisk -l

    Disk /dev/sda: 447.1 GiB, 480103981056 bytes, 937703088 sectors
    Disk model: KINGSTON SA400S3
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: gpt
    Disk identifier: 2E9821EF-2193-3E45-B7C3-6CDEEE505BAE

    Device       Start       End   Sectors   Size Type
    /dev/sda1     2048    206847    204800   100M EFI System
    /dev/sda2   206848   1026047    819200   400M Linux filesystem
    /dev/sda3  1026048 937703054 936677007 446.7G Linux filesystem

## Criando o luks
    cryptsetup luksFormat /dev/sda3
    cryptsetup luksOpen /dev/sda3 cryptroot

## Bom agora vamos no LVM
> **o live capaz de não ter, então:**

    apt update
    apt install lvm2

`lsblk`

## Primeiro o volume físico

    vgcreate vg00 /dev/mapper/cryptroot
    vgdisplay vg00

## Agora os volumes lógicos
    lvcreate -n raiz -L 33G vg00
    lvcreate -n home -l 100%FREE vg00

    lvdisplay
    lsblk

## Criando os sistemas de arquivos
    mkfs.vfat /dev/sda1
    mkfs.ext4 /dev/sda2
    mkfs.ext4 /dev/mapper/vg00-raiz
    mkfs.ext4 /dev/mapper/vg00-home

## Montando...
    mkdir /novo
    mount /dev/mapper/vg00-raiz /novo
    mkdir /novo/boot/
    mount /dev/sda2 /novo/boot

## Abrindo os tar
    cd /novo
    tar -xvf /media/user/externo/antigo-barra.tar

    mount /dev/mapper/vg-00-home /novo/home
    cd /novo/home
    tar -xvf /media/user/externo/antigo-home.tar

    cd /novo/boot
    tar -xvf /media/user/externo/antigo-boot.tar

`df -h`

    Filesystem            Type      Size  Used Avail Use% Mounted on
    udev                  devtmpfs  3.9G     0  3.9G   0% /dev
    tmpfs                 tmpfs     789M  9.5M  780M   2% /run
    /dev/sdb1             iso9660   2.4G  2.4G     0 100% /run/live/medium
    /dev/loop0            squashfs  2.1G  2.1G     0 100% /run/live/rootfs/filesystem.squashfs
    tmpfs                 tmpfs     3.9G  216M  3.7G   6% /run/live/overlay
    overlay               overlay   3.9G  216M  3.7G   6% /
    tmpfs                 tmpfs     3.9G   36M  3.9G   1% /dev/shm
    tmpfs                 tmpfs     5.0M  4.0K  5.0M   1% /run/lock
    tmpfs                 tmpfs     3.9G     0  3.9G   0% /sys/fs/cgroup
    tmpfs                 tmpfs     3.9G  504K  3.9G   1% /tmp
    tmpfs                 tmpfs     789M   40K  789M   1% /run/user/1000
    /dev/sdc              ext4      458G   81G  354G  19% /media/user/externo
    /dev/mapper/vg00-raiz ext4       33G   11G   20G  36% /novo
    /dev/sda2             ext4      380M  115M  241M  33% /novo/boot
    /dev/mapper/vg00-home ext4      407G   32G  355G   9% /novo/home

## Preparando para UEFI

    mkdir /novo/boot/efi
    mount /dev/sda1 /novo/boot/efi

## O chroot
    mount --bind /proc /novo/proc
    mount --bind /dev /novo/dev
    mount --bind /dev/pts /novo/dev/pts
    mount --bind /sys /novo/sys

`chroot /novo`

## Já no sistema
    apt update
    apt install arch-install-scripts

> **para ajudar no fstab**

    apt purge grub-common grub-pc grub-pc-bin grub2-common

    apt install grub-efi-amd64
    grub-install /dev/sda`

# Checando UUIDs
`blkid`

    /dev/sda1: SEC_TYPE="msdos" UUID="7232-729A" TYPE="vfat" PARTUUID="ebc9ff18-9cdb-f74a-9548-c7ff285f688a"
    /dev/sda2: UUID="cfb873df-5ae7-40a3-99d9-3867663ac099" TYPE="ext4" PARTUUID="72333b32-1864-4049-9563-b9ab39ee9b6e"
    /dev/sda3: UUID="696dbd1c-9fae-4604-b48b-6c66784bb810" TYPE="crypto_LUKS" PARTUUID="bd305056-a1c1-ba4f-910d-5db8e4229616"
    /dev/sdb1: UUID="2020-02-08-12-47-49-00" LABEL="d-live 10.3.0 ma amd64" TYPE="iso9660" PTUUID="60443c95" PTTYPE="dos" PARTUUID="60443c95-01"
    /dev/sdb2: SEC_TYPE="msdos" UUID="DEB0-0001" TYPE="vfat" PARTUUID="60443c95-02"
    /dev/loop0: TYPE="squashfs"
    /dev/mapper/cryptroot: UUID="81uBR5-6I1d-m50H-rcgd-0ncc-fe1V-dalNZQ" TYPE="LVM2_member"
    /dev/sdc: UUID="0e0a7f90-7bbb-4251-a073-bf26a2eefcf3" TYPE="ext4"
    /dev/mapper/vg00-raiz: UUID="31387c02-570d-4d4e-b47a-0bdb8fb8bef7" TYPE="ext4"
    /dev/mapper/vg00-home: UUID="4ed2eea6-d2d9-4c37-8cce-9b6b187fb36f" TYPE="ext4"


## Olhando a conf da cripto anterior
    cat /etc/crypttab

    sda5_crypt UUID=a43578ce-2511-47df-b4fd-2636f1b5e8e none luks,discard

## Preparando a nova
    echo "cryptroot UUID=696dbd-....4bb810 none luks,discard" > /etc/crypttab`

## Para o initrd poder ser capaz de decriptografar
    cp /usr/share/initramfs-tools/hooks/cryptkeyctl /etc/initramfs-tools/hooks

## Agora precisa editar o /etc/default/grub
    GRUB_CMDLINE_LINUX="root=/dev/mapper/cryptroot cryptdevice=UUID=696dbd1c-9fae-4604-b48b-6c66784bb810:cryptroot"`

    GRUB_ENABLE_CRYPTODISK=y

`update-initramfs -u -k all`
> **erros:**

    update-initramfs: Generating /boot/initrd.img-4.19.0-8-amd64
    **W: Couldn't identify type of root file system for fsck hook**
    W: Possible missing firmware /lib/firmware/nvidia/gv100/acr/ucode_load.bin for module nouveau
    update-initramfs: Generating /boot/initrd.img-4.19.0-6-amd64
    W: Couldn't identify type of root file system for fsck hook
    W: Possible missing firmware /lib/firmware/nvidia/gv100/acr/ucode_load.bin for module nouveau

Descobrimos depois que:

      cp /proc/mounts /mnt/etc/mtab

## Arruamando fstab

    genfstab -U / > /etc/fstab
    update-grub
    update-initramfs -u -k all

 ~~Errado:    GRUB_CMDLINE_LINUX="root=/dev/mapper/VG-00-raiz cryptdevice=UUID=696dbd1c-9fae-4604-b48b-6c66784bb810:cryptroot"~~

Corrigido: GRUB_CMDLINE_LINUX="root=/dev/mapper/vg00-raiz cryptdevice=UUID=696dbd1c-9fae-4604-b48b-6c66784bb810:cryptroot"

 **update-grub fica parado.**

 ~~Errado:    GRUB_CMDLINE_LINUX="root=/dev/mapper/VG-00-raiz cryptdevice=/dev/sda3:cryptroot"~~

Corrigido: GRUB_CMDLINE_LINUX="root=/dev/mapper/vg00-raiz cryptdevice=/dev/sda3:cryptroot"

**Bem, não vi nada diferente do que acho que deva ser o certo!
vamos nos preparar para o boot.**

    ctrl-d

    umount /novo/boot/efi
    umount /novo/boot/
    umount /novo/proc
    umount /novo/dev/pts
    umount /novo/dev
    umount /novo/sys
    umount /novo/home
    umount /novo

## Pequenos problemas para dar gosto!
hehe!!
pior que agora achei um artigo dizendo...
Update grub in a chroot environment with root on a luks encrypted volume
I looks like you forgot to create a proper /etc/mtap file

     cp /proc/mounts /mnt/etc/mtab

## Boot hd.
    grub>ls
    (proc) (hd0) (hd0,gpt3) (hd0,gpt2) (hd0,gpt1)`

    linux (hd0,gpt2)/vmlinuz-4.19.0-8-md64 root=/dev/mapper/vg00-raiz cryptdevice=/dev/sda3:cryptroot
    initrd (hd0,gpt2)/initrd.img-4.19.0-8-amd64

    GRUB_CMDLINE_LINUX="root=/dev/mapper/vg00-raiz cryptdevice=UUID=696dbd1c-9fae-4604-b48b-6c66784bb810:cryptroot"

    GRUB_CMDLINE_LINUX="root=/dev/mapper/vg00-raiz cryptdevice=/dev/sda3:cryptroot"

    linux (hd0,gpt2)/vmlinuz-4.19.0-8-md64 root=/dev/mapper/vg00-raiz cryptdevice=/dev/sda3:cryptroot init=/bin/bash
    initrd (hd0,gpt2)/initrd.img-4.19.0-8-amd64

## Bootado:
    mount -o remount,rw /

bom, então arruma o /etc/default/grub
trocando o VG pelo vg00

    mount -a
    update-grub
    genfstab -U / > /etc/fstab

    update-initramfs -u -k all

## Problema no .Xauthority
**Não lembrei como resolver fomos na ignorância**

       apt purge lightdm
       apt install lighdm`

## Tudo certo, até a próxima!

> **capaz que se resolva com:**
> 
    su - lightdm
    xauth generate :0 . trusted

## Boas dicas para serem testadas:
    insmod luks
    cryptomount (hd1,gpt3)
    set root=(crypto0)
    configfile (crypto)/boot/grub/grub.cfg

