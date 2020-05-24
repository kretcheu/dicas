# Criando um falso Live-cd

## Introdução

## Etapas

1. Criar um disco virtual
2. Criar as partições
3. Instalar o Debian
4. Instalar pacotes extras
5. Preparar o boot BIOS
6. Instalar um ambiente desktop
7. Copiando para um pendrive
8. Como usar


## Etapa 1 (Criar um disco virtual)
Primeiro vamos criar um arquivo para ser o nosso disco virtual.\
O objetivo é que desse modo você poderá usar o disco virtual para preparar um pendrive, para rodar numa máquina virtual ou distribuir para outros usuários.

```
dd if=/dev/zero of=disco-virtual bs=1M count=3072
ou
fallocate -l 3G disco-virtual
```

Criando um dispositivo virtual usando o arquivo *disco-virtual*.
```
losetup /dev/loop0 -P disco-virtual
```

Verificando:
```
fdisk -l /dev/loop0

Disk /dev/loop0: 3 GiB, 3221225472 bytes, 6291456 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

## Etapa 2 (Criar as partições)

### Criando novo particionamento
Como desejamos que o disco virtual sirva para dar boot em máquinas com UEFI, vamos criar um novo particionamento GPT.
```
fdisk /dev/loop0

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): g
Created a new GPT disklabel (GUID: 7B7598F2-48D3-2E4B-A1F7-FA69F933D733).
The old iso9660 signature will be removed by a write command.

Command (m for help): w

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

Vendo como ficou:
```
fdisk -l /dev/loop0

Disk /dev/loop0: 3 GiB, 3221225472 bytes, 6291456 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 471CE3D7-C134-3A49-A869-2729666C8626
```

Nossa tabela de partições terá 3 partições.
1. (1M)  Primeira para acomodar arquivos do GRUB para BIOS.
2. (50M) Para acomodar o GRUB para UEFI.
3. (~3G) Para os arquivos do Debian.

O Tamanho ideal da partição 3 depende do que pretende instalar, se for instalar um ambiente de desktop completo, 3Gb podem não ser suficientes.

### Criando as partições
```
fdisk  /dev/loop0

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): n
Partition number (1-128, default 1):
First sector (2048-6291422, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-6291422, default 6291422): +1M

Created a new partition 1 of type 'Linux filesystem' and of size 1 MiB.

Command (m for help): t
Selected partition 1
Partition type (type L to list all types): 4
Changed type of partition 'Linux filesystem' to 'BIOS boot'.

Command (m for help): n
Partition number (2-128, default 2):
First sector (4096-6291422, default 4096):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4096-6291422, default 6291422): +50M

Created a new partition 2 of type 'Linux filesystem' and of size 50 MiB.

Command (m for help): t
Partition number (1,2, default 2):
Partition type (type L to list all types): 1

Changed type of partition 'Linux filesystem' to 'EFI System'.

Command (m for help): n
Partition number (3-128, default 3):
First sector (106496-6291422, default 106496):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (106496-6291422, default 6291422):

Created a new partition 3 of type 'Linux filesystem' and of size 3 GiB.

Command (m for help): x
Expert command (m for help): A
Partition number (1-3, default 3): 1

The LegacyBIOSBootable flag on partition 1 is enabled now.

Expert command (m for help): r

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.

```

Vendo como ficou:
```
fdisk -l /dev/loop0

Disk /dev/loop0: 3 GiB, 3221225472 bytes, 6291456 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 471CE3D7-C134-3A49-A869-2729666C8626

Device        Start     End Sectors Size Type
/dev/loop0p1   2048    4095    2048   1M BIOS boot
/dev/loop0p2   4096  106495  102400  50M EFI System
/dev/loop0p3 106496 6291422 6184927   3G Linux filesystem
```

## Etapa 3 (Instalar o Debian)
Para instalar o Debian vamos usar uma ferramenta que automatiza o processo chamada **grml-debootstrap**.

### Instalando o grml-debootstrap
```
apt install grml-debootstrap
```

### Fazendo instalação
```
grml-debootstrap --release buster --efi /dev/loop0p2 --target /dev/loop0p3 --grub /dev/loop0 --hostname debianilo --password senharoot

 * EFI support enabled now.
 * grml-debootstrap [0.90] - Please recheck configuration before execution:

   Target:          /dev/loop0p3
   Install grub:    /dev/loop0
   Install efi:     /dev/loop0p2
   Using release:   buster
   Using hostname:  debianilo
   Using mirror:    http://deb.debian.org/debian
   Using arch:      amd64
   Config files:    /etc/debootstrap

   Important! Continuing will delete all data from /dev/loop0p3!

 * Is this ok for you? [y/N]
```

#### Ao digitar 'y' começa o processo...
```
Logical sector size is zero.
 * EFI partition /dev/loop0p2 doesn't seem to be formatted, creating filesystem.
mkfs.fat 4.1 (2017-01-24)
mkfs.fat: warning - lowercase labels might not work properly with DOS or Windows
 * Running mkfs.ext4  on /dev/loop0p3
mke2fs 1.45.5 (07-Jan-2020)
Discarding device blocks: done
Creating filesystem with 773115 4k blocks and 193536 inodes
Filesystem UUID: 43edb6ee-3109-44a5-843c-12f501a2fa76
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

 * No underlying block device for /dev/loop0p3 identified, skipping blockdev --rereadpt.
 * Disabling automatic filesystem check on /dev/loop0p3 via tune2fs
tune2fs 1.45.5 (07-Jan-2020)
Setting maximal mount count to -1
Setting interval between checks to 0 seconds
 * Mounting /dev/loop0p3 to /mnt/debootstrap.6361
 * Identified UUID 43edb6ee-3109-44a5-843c-12f501a2fa76 for /dev/loop0p3
 * Running debootstrap  for release buster (amd64) using http://deb.debian.org/debian
 * Executing: debootstrap --arch amd64   buster /mnt/debootstrap.6361 http://deb.debian.org/debian
I: Target architecture can be executed
I: Retrieving InRelease
I: Checking Release signature
I: Valid Release signature (key id 6D33866EDD8FFA41C0143AEDDCC9EFBF77E11517)
I: Retrieving Packages
I: Validating Packages
I: Resolving dependencies of required packages...
I: Resolving dependencies of base packages...
I: Checking component main on http://deb.debian.org/debian...
I: Retrieving libacl1 2.2.53-4
I: Validating libacl1 2.2.53-4
I: Retrieving adduser 3.118
I: Validating adduser 3.118
I: Retrieving libapparmor1 2.13.2-10
I: Validating libapparmor1 2.13.2-10
I: Retrieving apt 1.8.2
I: Validating apt 1.8.2
I: Retrieving apt-utils 1.8.2
I: Validating apt-utils 1.8.2

vários outros pacotes...

0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
   Executing stage remove_apt_cache
Cleaning apt cache.
   Executing stage services
[ ok ] Stopping OpenBSD Secure Shell server: sshd.
   Executing stage remove_chrootmirror
Finished chroot installation, exiting.
 * Removing chroot-script again
 * Unmounting bind-mount /run/udev
 * Unmount /mnt/debootstrap.6361
 * Removing /var/cache/grml-debootstrap/variables_loop0p3
 * Removing /var/cache/grml-debootstrap/stages_loop0p3
 * Finished execution of grml-debootstrap. Enjoy your Debian system.
```

## Etapa 4 (Instalar pacotes extras)
Terminada a instalação do Debian vamos "chrootar" o sistema novo instalado no disco virtual para instalar mais pacotes.

### Para chrootar
```
mount /dev/loop0p3 /mnt
mount /dev/loop0p2 /mnt/boot/efi

mount --bind /proc /mnt/proc
mount --bind /dev /mnt/dev
mount --bind /sys /mnt/sys

chroot /mnt

df -h

Filesystem      Size  Used Avail Use% Mounted on
/dev/loop0p3    2.9G  986M  1.8G  36% /
/dev/loop0p2     50M  130K   50M   1% /boot/efi
udev            1.9G     0  1.9G   0% /dev
```

Pronto, estamos rodando o sistema novo. Agora vamos instalar alguns outros pacotes e fazer algumas configurações.
Vamos agora para "pós-instalação".

#### Configurar o locales.
```
dpkg-reconfigure locales
```
Seleciono apenas **pt_BR.UTF-8 UTF-8 ** e deixo como default.

#### Criar um usuário.
```
adduser user
Adicionando usuário 'user' ...
Adicionando novo grupo 'user' (1000) ...
Adicionando novo usuário 'user' (1000) com grupo 'user' ...
Criando diretório pessoal '/home/user' ...
Copiando arquivos de '/etc/skel' ...
Nova senha:
Redigite a nova senha:
passwd: senha atualizada com sucesso
Modificando as informações de usuário para user
Informe o novo valor ou pressione ENTER para aceitar o padrão
	Nome Completo []:
	Número da Sala []:
	Fone de Trabalho []:
	Fone Residencial []:
	Outro []:
A informação está correta? [S/n]
```
Como pretendo distribuir a imagem do pendrive criei um usuário chamado *user* com senha *live* assim fica igual ao live-cd do Debian.

```
apt install sudo
```

#### Agora colocando o usuário no grupo sudo.
Vou colocar esse usuário no grupo *sudo* assim ele poderá ter privilégios de *root*.
```
adduser user sudo
Adicionando usuário 'user' ao grupo 'sudo' ...
Adicionando usuário user ao grupo sudo
Concluído.
```

#### Instalando alguns pacotes que considero importantes.
Você poderia ter alterado arquivos de configuração do *grml-debootstrap* para incluir pacotes automaticamente, mas como estou fazendo de forma genérica, preferi deixar para essa etapa.\
Sugiro que leia o manual para automatizar ainda mais o processo.

Eu não vou instalar softwares não-livres, como firmwares, mas dependendo do hardware que queira rodar pode ser que você precise.

```
apt install bash-completion network-manager shim-signed
```

#### Para configurar o mapa de teclado.
```
apt install console-setup
```

Eu selecionei o teclado brasileiro, mas você escolhe o que for mais conveniente.\
O teclado do meu Pc é japonês!

## Etapa 5 (Preparar o boot BIOS)

#### Para dar boot UEFI
Para o boot via UEFI rodar sem necessidade de configurações extras, usaremos um pequeno "truque".
```
mkdir /boot/efi/EFI/BOOT
cp  /boot/efi/EFI/debian/grubx64.efi /boot/efi/EFI/BOOT/BOOTX64.EFI
```

O disco virtual já é bootável usando UEFI, agora vamos prepará-lo para bootar também usando BIOS.
```
grub-install --target=i386-pc --recheck /dev/loop0
Instalando para a plataforma i386-pc.
Installation finished. No error reported.
```

### Recriando o grub.cfg
Vamos criar um grub.cfg que não coloque os sistemas que você porventura tenha na sua máquina.
```
chmod -x /etc/grub.d/30_os-prober
update-grub
```

## Etapa 6 (Instalar um ambiente desktop)
Eu escolhi instalar o ambiente Mate, mas claro, instale o da sua preferência, ou não instale nenhum, afinal ninguém é obrigado a usar ambiente gráfico!
```
apt install mate-desktop-environment lightdm

apt clean

```
Rodando `df -h` vemos quanto espaço ocupamos do hd-virtual:
```
df -h

Sist. Arq.      Tam. Usado Disp. Uso% Montado em
/dev/loop0p3    2,9G  2,5G  190M  94% /
/dev/loop0p2     50M  130K   50M   1% /boot/efi
udev            1,9G     0  1,9G   0% /dev
```

### Saindo do chroot
```
exit ou ctrl-d
```

Agora vamos ajustar o */etc/fstab*, pois em alguns casos o *grml-debootstrap* tem dificuldades.\
Quem faz muito bem esse papel é um programa chamado `genfstab` que está no pacote **arch-install-scripts**.\
Vamos instalar no seu Debian mesmo.
```
apt install arch-install-scripts

genfstab -U /mnt >/mnt/etc/fstab
```
Veja como ficou:
```
cat /mnt/etc/fstab

# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# /dev/loop0p3
UUID=43edb6ee-3109-44a5-843c-12f501a2fa76	/         	ext4      	rw,relatime	0 1

# /dev/loop0p2 LABEL=EFI\134x20System
UUID=414A-DE3E      	/boot/efi 	vfat      	rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro	0 2
```

Com tudo instalado e configurado, vamos desmontar as partições.

```
umount /mnt/proc
umount /mnt/dev
umount /mnt/sys
umount /mnt/boot/efi
umount /mnt/
```

## Etapa 7 (Copiando para um pendrive)
Como qualquer imagem de disco podemos fazer de várias formas, eu prefiro e recomendo o modo que acredito ser o mais simples e mais confiável.

Usando o programa `dd`
```
dd if=disco-virtual of=/dev/sdb bs=16M oflag=sync status=progress
```

Como provavelmente o tamanho do pendrive é maior que o arquivo do hd virtual haverá uma inconsistência na tabela de partições GPT.\
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

A partir daí pode criar outras partições ou estender o tamanho da partição 3.\
No artigo [Expandindo sistema de arquivos](expandindo.sistema.de.arquivos.md) tem instruções de como fazer.

## Conclusão
A ideia de usar um pen-drive com o Debian instalado é para dar flexibilidade de instalar mais pacotes e ir criando sua ferramenta ideal de trabalho.

Espero que tenha conseguido seguir os passos aqui apresentados.

Qualquer dúvida me procure no Telegram!

- [Grupo Debian Brasil](https://t.me/debianbrasil)
- [Grupo Debian BR](https://t.me/debianbr)


