## LVM / Luks

### Criando disco virtual

    fallocate -l 4G cripto

### Carregando módulo nbd

    modprobe nbd

### Associando a um dispositivo

    qemu-nbd -c /dev/nbd0 -f raw cripto 

### Criando partições

    fdisk /dev/nbd0 

Uma de 1G e outra com restante

### Criando dispositivo luks

    cryptsetup luksFormat --type luks2 /dev/nbd0p1
    ls -l /dev/mapper/

### Abrindo e mapeando dispositivo luks

    cryptsetup luksOpen /dev/nbd0p1 cripto1
    ls -l /dev/mapper/

### Criando dispositivo luks

    cryptsetup luksFormat --type luks2 /dev/nbd0p2
    cryptsetup luksOpen /dev/nbd0p2 cripto2

### Preparando Volumes

    apt install lvm2

### Criando volume físico

    vgcreate vg00 /dev/mapper/cripto1

    vgdisplay

### Criando volumes lógicos

    lvcreate -n raiz -L 500M vg00
    lvcreate -n home -l 100%FREE vg00

    pvs
    lvs
    vgs

### Montando
    mkfs.ext4 /dev/mapper/vg00-home
    mkfs.ext4 /dev/mapper/vg00-raiz


    lsblk

### Extendendo o volume lógico

    lvextend /dev/vg00/home /dev/mapper/cripto2

### Montando
    mount /dev/mapper/vg00-home /mnt/

### Verificando tamanho

    df -h

### Extndendo o sistema de arquivos

    resize2fs /dev/mapper/vg00-home

### Verificando tamanho

    df -h

### Desmontando tudo

    umount /mnt

### Desativando volume lógico

    vgchange -an vg00
    vgdisplay
    lsblk

### Fechando luks

    cryptsetup close cripto2
    cryptsetup close cripto1
    lsblk

### Desconectando do disco

    qemu-nbd -d /dev/nbd0

### Descarregando módulo

    modprobe -r nbd


