# Boot a partir de um disco virtual raw
A ideia desse tutorial é você usar um HD virtual para poder dar boot tanto como máquina virtual como na máquina real.

Para poder dar boot numa mâquina real usando um HD virtual raw

1. Preparar um HD virtual raw
2. Preparar o initrd
3. Ajustar o GRUB

## Etapa 1 (Preparar um HD virtual raw)
Vamos converter um HD virtual no formato qcow2 para *raw*.
```
qemu-img convert -O raw vdisk.qcow2 vdisk
```

## Etapa 2 (Preparar o initrd)
Dê boot na sua máquina virtual com o novo HD virtual.

### Incluir o script
**/etc/initramfs-tools/scripts/init-premount/vdisk** com o conteúdo:
```
#!/bin/sh
PREREQ="udev"
prereqs()
{
    echo "$PREREQ"
}

case "$1" in
    prereqs)
    prereqs
    exit 0
    ;;
esac

case "$device" in
   LABEL=* | UUID=* | PARTLABEL=* | PARTUUID=*)
   device="$(blkid -l -t "$device" -o device)" || return 1
          ;;
esac

if [ ! -z $vdisk ]; then
   mkdir /root2
   modprobe loop
   mount -n -t ext4 -o nodiratime,noatime $device /root2
   kpartx -av /root2$vdisk
fi
```

### Para incluir o binário do kpartx
Caso não tenha o pacote kpartx instalado, instale rodando:
```
apt install kpartx
```

Incluir o arquivo: **/etc/initramfs-tools/hooks/kpartx** com o conteúdo:
```
#!/bin/sh

set -e

PREREQ=""

prereqs () {
	echo "${PREREQ}"
}

case "${1}" in
	prereqs)
		prereqs
		exit 0
		;;
esac

. /usr/share/initramfs-tools/hook-functions

copy_exec /usr/sbin/kpartx /sbin

exit 0
```
### Dar permissão de execução aos scripts
```
chmod +x /etc/initramfs-tools/hooks/kpartx
chmod +x /etc/initramfs-tools/scripts/init-premount/vdisk
```

### Atualizar o initrd
```
update-initramfs -u -k all
```

### Anotando o UUID da partição do hd virtual
```
blkid

/dev/vda1: PARTUUID="e49b9379-6e54-ed48-a138-305681b629be"
/dev/vda2: SEC_TYPE="msdos" UUID="48A5-6E96" TYPE="vfat" PARTUUID="c548ba2a-f7a3-5d4d-897f-5b136b0a83e0"
/dev/vda3: UUID="032425af-fb6f-4844-b372-6098f78f064d" TYPE="ext4" PARTUUID="416bae8b-a525-b843-a4a1-9ebdd1e033e6"
```
Nesse exemplo o sistema está instalado na partição /dev/vda3.\
 Com o *blkid* consegue ver o UUID de cada partição.\
Anote o número para ser usado no GRUB, nesse caso: *032425af-fb6f-4844-b372-6098f78f064d*

Farei referência a ele como: **UUID-PART-VIRTUAL**

Agora pode desligar a máquina virtual.

## Etapa 3 (Ajustar o GRUB)
Na sua máquina real precisa preparar o GRUB para dar boot o HD virtual.

### Descobrir o UUID da partição real que tem o HD virtual
```
blkid

/dev/sda1: SEC_TYPE="msdos" UUID="1029-1F02" TYPE="vfat" PARTUUID="e9603a1e-fbe1-2149-adcf-3b2ac8ca0fe1"
/dev/sda2: UUID="336dbd2e-3e4f-4fba-9a01-6ea17e8b1802" TYPE="ext4" PARTUUID="24145314-33e9-c84f-8a09-41d8581046bb"
```
Nesse exemplo o sistema real está instalado na partição /dev/sda2.\
Com o *blkid* consegue ver o UUID de cada partição.\
Anote o número para ser usado no GRUB, nesse caso: *336dbd2e-3e4f-4fba-9a01-6ea17e8b1802*

Farei referência a ele como: **UUID-PART-REAL**

### Para incluir a entrada no GRUB
Edite o arquivo **/etc/grub.d/40_custom** com o conteúdo
```
menuentry "Vdisk" {
        # Removendo módulo tpm para contornar erro do loopback com UEFI
        rmmod tpm
        insmod part_gpt
        loopback loop (hd0,gpt2)/vdisk
        linux (loop,gpt3)/boot/vmlinuz-5.4.0-4-amd64 root=UUID=032425af-fb6f-4844-b372-6098f78f064d device=UUID=336dbd2e-3e4f-4fba-9a01-6ea17e8b1802 vdisk=/vdisk 
        initrd (loop,gpt3)/boot/initrd.img-5.4.0-4-amd64
}
```

Na linha **loopback** deve indicar o disco, partição e nome do arquivo do hd-virtual.

Na linha **linux** precisa passar os parâmetros:

- root= -> apontando para a partição no hd-virtual **UUID-PART-VIRTUAL**
- device -> apontando para a partição no HD real que contém o disco virtual. **UUID-PART-REAL**
- vdisk -> indicando o nome do disco virtual. Usei /vdisk porque o disco virtual está o diretório raiz.

Para criar o novo arquivo de configuração do grub com essa entrada rode:
```
update-grub
```

Agora dê boot na máquina real e teste se está tudo ok.

Espero que tenha conseguido seguir os passos aqui apresentados.

## Adaptações para o Parábola e derivados do Arch
Agradecimento ao amigo @Simplex pelas dicas!

### Instalar kpartx
```
pacman -S multipath-tools
```

### Arquivos que precisa criar:

1. /etc/initcpio/install/vdisk

Com o conteúdo:
```
#!/bin/bash

build() {
        add_runscript
}

help() {
        echo 'Incluindo script vdisk'
}
```

2. /etc/initcpio/hooks/vdisk

Com o conteúdo:

```
#!/usr/bin/ash

run_earlyhook() {
    case "$device" in
       LABEL=* | UUID=* | PARTLABEL=* | PARTUUID=*)
       device="$(blkid -l -t "$device" -o device)" || return 1
              ;;
    esac

    if [ ! -z $vdisk ]; then
       mkdir /root2
       modprobe loop
       mount -n -t ext4 -o nodiratime,noatime $device /root2
       kpartx -avn /root2$vdisk
    fi
}

```

### Arquivos que precisa editar

1. Edite o arquivo /etc/mkinitcpio.conf

- na linha MODULES inclua loop e dm-mod

exemplo:
```
MODULES=(loop dm-mod)
```

- na linha BINARIES inclua kpartx
exemplo:
```
BINARIES=(kpartx)
```

- na linha HOOKS inclua vdisk
exemplo:
```
HOOKS=(base udev autodetect modconf block filesystems keyboard fsck vdisk)
```

### Criando o initram
```
mkinitcpio -P
```

### Ajuste do grub na distro principal

Recrie o arquivo de configuração do grub, colocando a entrada em:
/etc/grub.d

Se usar Parábola ou derivado do Arch rode:

```
grub-mkconfig -o /boot/grub/grub.cfg
```

### Qualquer dúvida me procure no Telegram!

- [Grupo Gurso GNU](https://t.me/cursognu)
- [Grupo Debian Brasil](https://t.me/debianbrasil)
- [Grupo Debian BR](https://t.me/debianbr)

