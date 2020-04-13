# Transplante Debian com Luks (máquina BIOS)

## Introdução
Lucks é uma especificação de criptografia de disco "Linux Unified Key Setup".\
Nesse tutorial vamos transplantar uma instação de Debian implementando Luks.

O objetivo é que mesmo que alguém tenha acesso físico a sua máquina não consiga ter acesso aos seus dados.\
O método que vamos usar será para criptografar todo o disco, numa máquina que use BIOS e sem que você precise reinstalar o Debian do zero.


## Descrição das etapas

Vamos fazer os seguintes passos:

 1. Baixar a imagem de um live-cd.
 2. Preparar um pen-drive bootável.
 3. Dar boot pelo pen-drive.
 4. Fazer um backup num hd externo.
 5. Remodelar o particionamento do disco.
 6. Implementar a criptografia luks.
 7. Criar os sistemas de arquivos (formatar).
 8. Restaurar os arquivos do Debian.
 9. Preparar para poder bootar.
 10. Bootar o sistema transplantado.

## Executando cada etapa.

### Etapa 1 (Download imagem ISO)
Acesse o site de download de imagens do Debian, para escolher qual imagem usar.\
Há imagens de live-cd para arquitetura i386, que são para computadores com processadores de 32bits e imagens amd64 para computadores de 64bits.

Imagens para 32 bits: [i386][debian-i386]\
Imagens para 64 bits: [amd64][debian-amd64]

Vou usar como exemplo o live-cd com Mate, mas escolha o da sua preferência.\
Rodando seu Debian, abra um terminal para baixar o arquivo iso e os hashs.
```
# Para amd64
wget https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/debian-live-10.3.0-amd64-mate.iso
wget https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/SHA1SUMS

# Para i386
wget https://cdimage.debian.org/debian-cd/current-live/i386/iso-hybrid/debian-live-10.3.0-i386-mate.iso
wget https://cdimage.debian.org/debian-cd/current-live/i386/iso-hybrid/SHA1SUMS
```
Completado download vamos verificar a integridade do arquivo de imagem iso.
```
# Para i386 e amd64
sha1sum -c SHA1SUMS --ignore-missing
```

### Etapa 2 (Preparar pen-drive)
Com a imagem iso verificada vamos preparar um pen-drive para dar boot por ele.\
Há vários programas que fazem isso, e de vários modos, eu vou usar o que penso ser o mais confiável e fácil.

Separe um pen-drive de 4Gb que será usado **exclusivamente** para isso, ou seja, os dados nele serão sobrescritos.\
Insira o pen-drive numa porta usb, quando o pendrive já está com dados é comum que suas partições já sejam montadas automaticamente.\
Primeiro precisamos saber o nome de dispositivo que foi atribuido a ele.
```
fdisk -l

Disk /dev/sdb: 3,77 GiB, 4023386112 bytes, 7858176 sectors
Disk model: USB Flash Disk
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x38470cfe

Device     Boot Start     End Sectors  Size Id Type
/dev/sdb1  *        0 7082879 7082880  3,4G  0 Empty
/dev/sdb2       21332   22163     832  416K ef EFI (FAT-12/16/32)
```

Nesse caso o dispositivo é `/dev/sdb`, vamos desmontar as partições caso tenham sido montadas.\
Faça os devidos ajustes nos nomes dos dispositivos para o seu caso.
```
umount /dev/sdb1
umount /dev/sdb2
```
Desmontadas as partições vamos gravar a imagem no pen-drive.
```
# Para amd64
dd if=debian-live-10.3.0-amd64-mate.iso of=/dev/sdb bs=16M oflag=sync status=progress

# Para i386
dd if=debian-live-10.3.0-i386-mate.iso of=/dev/sdb bs=16M oflag=sync status=progress
```

### Etapa 3 (Boot pen-drive)
Para dar boot pelo pen-cdrive vai reiniciar o PC e será preciso indicar a BIOS que quer dar boot por ele.\
Dependendo do modelo de BIOS há uma tecla a ser apertada para selecionar a partir de qual dispositivo será feito o boot.\
Verifique qual é a tecla usada no seu PC, é comum ser F8.\
Caso não saiba pode também usar a tecla para o *setup da BIOS* normalmente f2 ou esc.

Definido o dispositivo de boot, o boot começa e é apresentado uma tela do bootloader indicando algumas possibilidades:

- Debian GNU/Linux Live
- Debian Live Localisation Support
- Graphical Debian Installer
- Debian Installer

Vamos escolher a primeira **Debian GNU/Linux Live** para ter um sistema rodando e podermos seguir.

### Etapa 4 (Fazer o Backup)
Com o Debian Live rodando vamos abrir um terminal e obter privilégios de root.
```
user@debian:~$ sudo -s
root@debian:/home/user#
```
Agora vamos montar uma partição do HD externo e a partição ou partições do nosso Debian.

#### Criando os pontos de montagem
```
mkdir /debian
mkdir /hd-externo
```
#### Montando as partições
Claro que vai precisar saber qual os nomes de dispositivos e quais as partições que vai montar.\
Nesse exemplo temos:
```
mount /dev/vda1 /debian
mount /dev/vdb1 /hd-externo/
```
#### Criando o backup com tar
```
cd /debian
tar -cvf /hd-externo/backup.debian.tar *
```

#### Desmontando sistema antigo
Uma vez que o backup está feito podemos desmontar a partição que estava o sistema antigo.
```
umount /debian
```

### Etapa 5 (Particionar o disco)
Antes de colocar os comandos para os programas rodarem, precisamos pensar em qual vai ser nossa estrutura de particionamento.\
Nessa etapa você pode pensar no melhor particionamento que desejar, independente de como ele está atualemnte no seu sistema.

No nosso exemplo vou usar 2 partições, uma para o GRUB e outra para o restante do sistema.\
Veja, não é todo o `/boot` que vai ficar nessa partição, apenas os arquivos do GRUB.

Essa estrutura visa não expor o kernel e o initrd para acesso não autorizado, por quem tenha acesso físico a máquina.

Usando fdisk vamos apagar as partições existentes e criar a nova estrutura.
```
fdisk /dev/vda

Welcome to fdisk (util-linux 2.33.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): p
Disk /dev/vda: 7 GiB, 7516192768 bytes, 14680064 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x64683414

Device     Boot  Start      End  Sectors Size Id Type
/dev/vda1         2048   104447   102400  50M 83 Linux
/dev/vda2       104448 14680063 14575616   7G 83 Linux

Command (m for help): d
Partition number (1,2, default 2):

Partition 2 has been deleted.

Command (m for help): d
Selected partition 1
Partition 1 has been deleted.

Command (m for help): p
Disk /dev/vda: 7 GiB, 7516192768 bytes, 14680064 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x64683414

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-14680063, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-14680063, default 14680063): +50M

Created a new partition 1 of type 'Linux' and of size 50 MiB.
31mPartition #1 contains a ext4 signature.

Do you want to remove the signature? [Y]es/[N]o: Y

The signature will be removed by a write command.

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2):
First sector (104448-14680063, default 104448):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (104448-14680063, default 14680063):

Created a new partition 2 of type 'Linux' and of size 7 GiB.
31mPartition #2 contains a crypto_LUKS signature.

Do you want to remove the signature? [Y]es/[N]o: Y

The signature will be removed by a write command.

Command (m for help): p
Disk /dev/vda: 7 GiB, 7516192768 bytes, 14680064 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x64683414

Device     Boot  Start      End  Sectors Size Id Type
/dev/vda1         2048   104447   102400  50M 83 Linux
/dev/vda2       104448 14680063 14575616   7G 83 Linux

Filesystem/RAID signature on partition 1 will be wiped.
Filesystem/RAID signature on partition 2 will be wiped.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```
Para evitar problemas vamos fazer com que o kernel releia a tabela de partições:
```
partprobe
```

Ficou assim:
```
fdisk /dev/vda -l

Disk /dev/vda: 7 GiB, 7516192768 bytes, 14680064 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x64683414

Device     Boot  Start      End  Sectors Size Id Type
/dev/vda1         2048   104447   102400  50M 83 Linux
/dev/vda2       104448 14680063 14575616   7G 83 Linux
```

### Etapa 6 (Implementar a criptografia luks)
Vamos preparar a partição para poder usar Luks, estamos usando a versão 1 (**luks1**), pois o GRUB ainda não consegue abrir a versão 2.\
Será necessário fornecer uma "passphrase", escolha uma complexa, não adianta nada usar luks com uma passphrase 123, ou qualquer outra facilmente dedutível.

```
cryptsetup luksFormat --type luks1 /dev/vda2

WARNING!
========
This will overwrite data on /dev/vda2 irrevocably.

Are you sure? (Type uppercase yes): YES
Enter passphrase for /dev/vda2:
Verify passphrase:
```

Agora vamos criar um mapeamento para partição Luks.
```
cryptsetup luksOpen /dev/vda2 croot

lsblk

NAME      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
vda       254:0    0    7G  0 disk
├─vda1    254:1    0   50M  0 part
└─vda2    254:2    0    7G  0 part
  └─croot 253:0    0    7G  0 crypt
vdb       254:16   0    6G  0 disk
└─vdb1    254:17   0    6G  0 part  /hd-externo
```

### Etapa 7 (Criar os sistemas de arquivos)

```
mkfs.ext4 /dev/vda1
mkfs.ext4 /dev/mapper/croot
```

### Etapa 8 (Restaurar os arquivos do Debian)
#### Montando
```
mount /dev/mapper/croot /debian/
mkdir -p /debian/boot/grub
mount /dev/vda1 /debian/boot/grub
```

#### Restaurar o backup
```
cd /debian/
tar -xvf /hd-externo/backup.debian.tar
```

### Etapa 9 (Preparar para poder bootar)
#### Preparar o /etc/fstab
```
apt install arch-install-scripts

genfstab -U /debian/ /debian/etc/fstab

cat /debian/etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# /dev/mapper/croot
UUID=d0593f81-017f-4c46-8b6a-242d9e3fe0b6       /               ext4            rw,relatime     0 1

# /dev/vda1
UUID=ae2c2916-c853-4f54-b2e4-fbaaa168db74       /boot/grub      ext4            rw,relatime     0 2
```

#### Preparar para o chroot
```
cd /debian

mount --bind /proc proc
mount --bind /dev dev
mount --bind /sys sys

chroot .
```
obs.: Agora já é no sistema novo!

```

cat /etc/fstab 
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# /dev/mapper/croot
UUID=d0593f81-017f-4c46-8b6a-242d9e3fe0b6	/         	ext4      	rw,relatime	0 1

# /dev/vda1
UUID=ae2c2916-c853-4f54-b2e4-fbaaa168db74	/boot/grub	ext4      	rw,relatime	0 2

```

#### Arquivo de configuração do GRUB
Vamos usar o UUID da partição na configuração do GRUB.

Veja, não é o UUID do Lucks é o UUID da partição, rodando blkid você pode verificar:
```
blkid 

/dev/vda1: UUID="ae2c2916-c853-4f54-b2e4-fbaaa168db74" TYPE="ext4" PARTUUID="64683414-01"
/dev/vda2: UUID="a50a9263-bdd5-4f8d-941d-0b6c883e1bcf" TYPE="crypto_LUKS" PARTUUID="64683414-02"
/dev/mapper/croot: UUID="d0593f81-017f-4c46-8b6a-242d9e3fe0b6" TYPE="ext4"

```
Edite o arquivo `/etc/default/grub` incluindo as duas seguintes linhas, faça a adaptação para o UUID que recebeu a partição.
```
GRUB_CMDLINE_LINUX="root=/dev/mapper/croot cryptdevice=UUID=a50a9263-bdd5-4f8d-941d-0b6c883e1bcf:croot"
GRUB_ENABLE_CRYPTODISK=y
```

#### Instalando o programa cryptsetup
Nesse momento você precisa de conectividade com Internet, caso ainda não esteja conectado, conecte-se agora.

Primeiro vamos atualizar a lista de pacotes.
```
aps update
apt install cryptsetup
```
#### Criando uma chave e incluindo num slot
Para o kernel ser capaz de decriptografar a partição "/" sem que seja necessário digitar duas vezes a passphrase, uma para o GRUB outra para o kernel, vamos colocar uma chave no slot luks.
```
dd bs=512 count=4 if=/dev/urandom of=/crypto_keyfile.bin
4+0 records in
4+0 records out
2048 bytes (2.0 kB, 2.0 KiB) copied, 0.00128756 s, 1.6 MB/s

chmod 000 /crypto_keyfile.bin
cryptsetup luksAddKey /dev/vda2 /crypto_keyfile.bin

```
#### Vendo como ficou

```
cryptsetup luksDump /dev/vda2
LUKS header information for /dev/vda2

Version:        1
Cipher name:    aes
Cipher mode:    xts-plain64
Hash spec:      sha256
Payload offset: 4096
MK bits:        512
MK digest:      9e f8 5d 17 2e cf 4e 18 91 db e7 e1 79 20 b6 9c 37 84 c2 0e
MK salt:        86 a1 dd 8e 94 10 2f af 84 8a 48 ce e7 22 95 31
                41 60 e0 a8 96 56 a2 0c ca 9f 45 96 49 6e c8 e0
MK iterations:  73306
UUID:           a50a9263-bdd5-4f8d-941d-0b6c883e1bcf

Key Slot 0: ENABLED
        Iterations:             1168980
        Salt:                   e0 c0 9c 8a f0 28 18 57 7a 52 63 c1 ec bd 66 a6
                                35 67 7c 59 13 dd 90 fa 44 e5 18 67 58 b2 1f 7e
        Key material offset:    8
        AF stripes:             4000
Key Slot 1: ENABLED
        Iterations:             1021008
        Salt:                   6e 7d 90 b2 fa db c0 2f 65 f2 74 ae 94 4d a9 62
                                76 a4 ae b1 42 05 f7 09 90 08 0f 09 57 8d f5 3f
        Key material offset:    512
        AF stripes:             4000
Key Slot 2: DISABLED
Key Slot 3: DISABLED
Key Slot 4: DISABLED
Key Slot 5: DISABLED
Key Slot 6: DISABLED
Key Slot 7: DISABLED
```

#### Editando /etc/crypttab
Edite o arquivo `/etc/crypttab` incluindo a partição luks, adapte para o UUID que recebeu a partição.
```
# <target name> <source device>         <key file>      <options>
croot UUID=a50a9263-bdd5-4f8d-941d-0b6c883e1bcf /crypto_keyfile.bin luks,discard
```

#### Editando /etc/cryptsetup-initramfs/conf-hook
Edite o arquivo `/etc/cryptsetup-initramfs/conf-hook` acrescentando as linhas abaixo para que o gerador do initram inclua a chave e os programas necessários.
```
CRYPTSETUP=y
KEYFILE_PATTERN=/*.bin
```
#### Editando /etc/initramfs-tools/initramfs.conf
Para que um usuârio que não seja root não possa ler o initrd
```
UMASK=0077
```

### Preparando initrd para poder abrir luks
```
cp /usr/share/initramfs-tools/hooks/cryptkeyctl /etc/initramfs-tools/hooks
```

#### Resume
Como nesse caso não estamos usando swap vamos desabilitar o "resume" que é um recurso necessário para hibernação.
```
echo "RESUME=none" >/etc/initramfs-tools/conf.d/resume
```

#### Atualizando initrd
Para que os recursos de criptografia estejam disponíveis precisamos atualizar o initrd, faremos isso para todas as versões de kernel que tiver instalado.
```
update-initramfs -u -k all
update-initramfs: Generating /boot/initrd.img-4.19.0-8-amd64
```

#### Instalando grub
Para que o sistema transplantado possa bootar vamos instalar o GRUB no disco.
```
grub-install /dev/vda
Installing for i386-pc platform.
Installation finished. No error reported.
```

#### Recriando arquivo de configuração do grub
Agora vamos regerar o arquivo grub.cfg que irá incluir as alterações de UUID bem como as configuração para criptografia.
```
update-grub

Generating grub configuration file ...
Found background image: /usr/share/images/desktop-base/desktop-grub.png
Found linux image: /boot/vmlinuz-4.19.0-8-amd64
Found initrd image: /boot/initrd.img-4.19.0-8-amd64
done
```

### Etapa 10
#### Saindo do chroot e desmonstando
Trabalho feito, agora vamos sair do chroot para poder dar boot no nosso Debian transplantado.
```
exit

umount proc
umount dev
umount sys
umount boot/grub
cd ..
umount /debian

```
#### Dando reboot:
```
shutdown -r now
```

### Caso tenha problemas (troubleshooting)
Caso seu sistema não dê boot, pode usar o console do GRUB para dar boot e resolver o problema.\
Se não conseguir ver a tela do Grub, pode usar o GRUB do live-cd do Debian.\
Dê boot pelo pen-drive como fez da primeira vez.

Na tela do GRUB tecle *c* para usar o shell do Grub.

rode: `ls` para ver os nomes dos discos.\
Você vai precisar saber qual é o do seu sistema.

Para montar decriptografando, use os comandos abaixo, será necessário digitar a passphrase que definiu para o luks.
```
insmod luks
cryptomount (hd1,msdos2)
configfile (crypto0)/boot/grub/grub.cfg
```
Agora na tela do grub do seu sistema novo.\
Dê boot, logue como root, para verificar o que pode ter dado errado.

- Algumas coisas a verificar:
```
lsblk
cat /etc/crypttab
```

- Compare se está correto o UUID ou o nome do dispositivo que usou.


- Verifique o /etc/fstab.
- Verifique se o nome da chave está correto no arquivo /etc/crypttab: `/crypto_keyfile.bin`
- Verifique se está no slot do luks.
```
cryptsetup luksDump /dev/vdc2
```

### Conclusão
A "nova" instalação está agora mais protegida, mesmo que alguém tenha acesso físico a sua máquina, não conseguirá ter acesso ao seus dados, nem mesmo rodar seu sistema.

No entando ainda é vulnerável ao "Ataque da diabólica camareira"\
[to Evil Maid attacks.](https://www.schneier.com/blog/archives/2009/10/evil_maid_attac.html)



[debian-amd64]: https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/
[debian-i386]: https://cdimage.debian.org/debian-cd/current-live/i386/iso-hybrid/
