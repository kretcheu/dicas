# Criar um pendrive apenas com GRUB
A ideia desse tutorial é criar um pendrive que seja capaz de carregar o GRUB na maior quantidade de dispositivos possîveis.\
Muitas vezes é necessário para resolver problemas em máquinas com dual-boot.

Nesse pendrive colocaremos apenas o bootloader, sem nenhum sistema.\

1. Etapa 1 (Criar um disco virtual)
2. Etapa 2 (Criar as partições)
3. Etapa 3 (Criar o sistema de arquivos)
4. Etapa 4 (Instalar pacotes necessários)
5. Etapa 5 (Copiar os arquivos)
6. Etapa 6 (criar grub.cfg)
7. Etapa 7 (Copiando para um pendrive)


## Etapa 1 (Criar um disco virtual)
Primeiro vamos criar um arquivo para ser o nosso disco virtual.\
O objetivo é que desse modo você poderá usar o disco virtual para preparar o pendrive ou distribuir para outros usuários.

```
dd if=/dev/zero of=disco-virtual bs=1M count=25
```

Criando um dispositivo virtual usando o arquivo *disco-virtual*.
```
losetup /dev/loop1 -P disco-virtual
```

Verificando:
```
fdisk -l /dev/loop1

Disk /dev/loop1: 20 MiB, 20971520 bytes, 40960 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

## Etapa 2 (Criar as partições)

### Criando novo particionamento
Como desejamos que o disco virtual sirva para carregar o GRUB em máquinas com BIOS e UEFI, vamos criar um novo particionamento GPT.
```
fdisk /dev/loop1

Welcome to fdisk (util-linux 2.33.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xbaa87b08.

Command (m for help): g
Created a new GPT disklabel (GUID: 4EA349C4-18B8-3548-8681-063867BA0046).

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

Vendo como ficou:
```
fdisk -l /dev/loop1

Disk /dev/loop1: 20 MiB, 20971520 bytes, 40960 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 4EA349C4-18B8-3548-8681-063867BA0046
```

Nossa tabela de partições terá 2 partições.
1. (1M)  Para acomodar arquivos do GRUB para BIOS.
2. (19M) Para acomodar o GRUB UEFI e os módulos das duas versões.

### Criando as partições
```
fdisk  /dev/loop1

Welcome to fdisk (util-linux 2.33.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): n
Partition number (1-128, default 1):
First sector (2048-40926, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-40926, default 40926): +1M

Created a new partition 1 of type 'Linux filesystem' and of size 1 MiB.

Command (m for help): t
Selected partition 1
Partition type (type L to list all types): 4
Changed type of partition 'Linux filesystem' to 'BIOS boot'.

Command (m for help): n
Partition number (2-128, default 2):
First sector (4096-40926, default 4096):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4096-40926, default 40926):

Created a new partition 2 of type 'Linux filesystem' and of size 18 MiB.

Command (m for help): t
Partition number (1,2, default 2):
Partition type (type L to list all types): 1

Changed type of partition 'Linux filesystem' to 'EFI System'.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

Veja como ficou:
```
fdisk -l /dev/loop1
Disk /dev/loop1: 20 MiB, 20971520 bytes, 40960 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 4EA349C4-18B8-3548-8681-063867BA0046

Device       Start   End Sectors Size Type
/dev/loop1p1  2048  4095    2048   1M BIOS boot
/dev/loop1p2  4096 40926   36831  18M EFI System
```

Agora uma parte fundamental para poder dar boot em máquinas apenas com BIOS.
```
fdisk /dev/loop1

Welcome to fdisk (util-linux 2.33.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): x

Expert command (m for help): A
Partition number (1,2, default 2): 1

The LegacyBIOSBootable flag on partition 1 is enabled now.

Expert command (m for help): r

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

## Etapa 3 (Criar o sistema de arquivos)
Vamos criar o sistema de arquivos (formatar) e montar a partição.
```
mkfs.vfat /dev/loop1p2
mkfs.fat 4.1 (2017-01-24)

mount /dev/loop1p2 /mnt
```

## Etapa 4 (Instalar pacotes necessários)
Para podermos colocar as duas versões de GRUB, precisamos dos pacotes correspondentes a cada uma.
```
apt install grub-efi-amd64-bin grub-pc-bin grub2-common
```

## Etapa 5 (Copiar os arquivos)
```
grub-install --target=x86_64-efi --efi-directory=/mnt/ --boot-directory=/mnt/boot --removable /dev/loop1
grub-install --target=i386-pc --recheck --boot-directory=/mnt/boot /dev/loop1

cp /usr/share/desktop-base/futureprototype-theme/grub/grub-16x9.png /mnt/boot/grub
```

## Etapa 6 (criar grub.cfg)
Edite o arquvivo **/mnt/boot/grub/grub.cfg** com o conteúdo:

```
function load_video {
  if [ x$feature_all_video_module = xy ]; then
    insmod all_video
  else
    insmod efi_gop
    insmod efi_uga
    insmod ieee1275_fb
    insmod vbe
    insmod vga
    insmod video_bochs
    insmod video_cirrus
  fi
}

if [ x$feature_default_font_path = xy ] ; then
   font=unicode
else
    font="$prefix/unicode.pf2"
fi

if loadfont $font ; then
  set gfxmode=1024x768x32
  load_video
  insmod gfxterm
  set locale_dir=$prefix/locale
  set lang=pt_BR
  insmod gettext
fi
terminal_input at_keyboard
terminal_output gfxterm

insmod png

if background_image $prefix/grub-16x9.png; then
  set color_normal=white/black
  set color_highlight=black/white
else
  set menu_color_normal=cyan/blue
  set menu_color_highlight=white/blue
fi

if [ ${grub_platform} == "pc" ]; then
  set color_normal=yellow/black
  set color_highlight=black/yellow
  set menu_color_normal=yellow/black
  set menu_color_highlight=black/yellow
fi

function gfxmode {
	set gfxpayload="${1}"
}
set linux_gfx_mode=
export linux_gfx_mode

menuentry "Detectar grub.cfg..."  {
    insmod part_gpt
    insmod part_msdos
    insmod fat
    insmod ext2

             for efi in (*,gpt*)/*/grub.conf (*,gpt*)/*/*/grub.cfg (*,msdos*)/*/grub.conf (*,msdos*)/*/*/grub.conf ; do
                regexp --set=1:efi_device '^\((.*)\)/' "${efi}"
                if [ -e "${efi}" ]; then
                    efi_found=true

                    menuentry --class=grub "${efi}" "${efi_device}" {
                        root="${2}"
                        configfile "${1}"
                    }
                fi
            done

            if [ "${efi_found}" != true ]; then
                menuentry --hotkey=q --class=find.none "No GRUB files detected." {menu_reload}
            else
                menuentry --hotkey=q --class=cancel "Cancel" {menu_reload}
            fi
    }
}

```

### Desmontando
```
sync
umount /mnt
```

## Etapa 7 (Copiando para um pendrive)
Como qualquer imagem de disco podemos fazer de várias formas, eu prefiro e recomendo o modo que acredito ser o mais simples e mais confiável.

Usando o programa `dd`
```
dd if=disco-virtual of=/dev/sdb bs=1M oflag=sync status=progress

```

Como certamente o tamanho do pendrive é BEM maior que o arquivo do hd virtual haverá uma inconsistência na tabela de partições GPT.\
Vamos corrigir isso usando o `fdisk`.

Ao rodar o programa aparecerá uma mensagem de erro, em virtude da tabela de partições GPT.

**GPT PMBR size mismatch (6291455 != 15633407) will be corrected by write.\
The backup GPT table is not on the end of the device. This problem will be corrected by write.
**

Para corrigir, basta rodar:
```
fdisk  /dev/sdb

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

A partir daí pode criar outras partições.\
No artigo [Expandindo sistema de arquivos](expandindo.sistema.de.arquivos.md) tem instruções de como fazer.

## Conclusão
A ideia de usar um pen-drive com apenas o GRUB instalado é para ter uma ferremanta que ajuda muito caso o sistema não esteja dando boot por falta ou problema no bootloader.

Espero que tenha conseguido seguir os passos aqui apresentados.

O que eu fiz pode ser baixado no link [disco-virtual.img](https://github.com/kretcheu/download/blob/master/disco-virtual.img?raw=true)
Qualquer dúvida me procure no Telegram!

- [Grupo Debian Brasil](https://t.me/debianbrasil)
- [Grupo Debian BR](https://t.me/debianbr)



