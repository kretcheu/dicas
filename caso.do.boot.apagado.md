# Caso do /boot apagado.

## Histórico
Ao fazer uma manutenção nosso amigo confundiu um disco externo com o HD interno da máquina.   
Por engano apagou duas partições, criou outra, criou sistema de arquivos, gravou dados, no entanto essas duas partições eram as usadas pelo Debian.  
Não havia como recuperar diretamente os dados que um dia foram usados pelo Debian para o `/boot` e `/boot/efi`.
   
Como o restante do sistema estava integro, eu lancei o **"Reinstalar Jamais!"**, propondo que poderíamos resolver a bronca sem "formatações" e a reinstalação do Debian.   
Ele topou e aqui descrevo o que fizemos para salvar mais esse Debian.

## Usando um live-cd do Debian
Para decriptografar e criar o disposito da partição

    cryptsetup luksOpen /dev/sda3 sda3_crypt

## para ver o LVM
    apt install lvm2
    lvdisplay
    vgchange -ay vg_server

## para ativar o volume
    vgchange -ay debian-vg

## para montar o / (barra)
    mount  /dev/debian-vg/root /mnt

## Usando fdisk recriamos 2 partições para colocar /boot e /boot/efi
    fdisk /dev/sda
    mkfs.vfat /dev/sda1
    mkfs.ext4 /dev/sda2

## para montar o que falta para o chroot
    mount -t proc proc /mnt/proc
    mount -o bind /dev /mnt/dev
    mount -o bind /sys /mnt/sys
    mount -o bind /dev/pts /mnt/dev/pts

    mount /dev/sda2 /mnt/boot
    mount /dev/sda1 /mnt/boot/efi

## para chrootar
    chroot /mnt

## Reinstalar os pacotes que usam /boot
    apt update
    apt reinstall linux-image-amd64
    apt install grub-efi-amd64
    update-initramfs -u -k all

## Saindo do chroot
   ctrl-d

## para desmontar tudo
    umount /mnt/boot/efi
    umount /mnt/boot/
    umount /mnt/dev/pts
    umount /mnt/dev/
    umount /mnt/proc
    umount /mnt/sys
    umount /mnt

## arquivos de configuração
    /etc/default/grub
    GRUB_CMDLINE_LINUX="root=/dev/mapper/debian--vg-root cryptdevice=/dev/sda3:sda3_crypt"

    /etc/crypttab
    sda3_crypt UUID=d4296d05-9fc9-4e6f-966a-0df4647cd6a6 none luks


