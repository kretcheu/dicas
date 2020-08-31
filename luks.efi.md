# Transplantando instalação e implementando Lucks

## A máquina tem 2 HDs
```
fdisk -l
Disk /dev/loop0: 3 GiB, 3221225472 bytes, 6291456 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/vda: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2EA7B5C3-1854-0C47-950B-C48585611FC1

Device      Start      End  Sectors Size Type
/dev/vda1    2048   104447   102400  50M EFI System
/dev/vda2  104448 41943006 41838559  20G Linux filesystem

Disk /dev/vdc: 6 GiB, 6442450944 bytes, 12582912 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 5748A7EF-C172-764A-B59F-6F414AF76423
```

## Preparando o segundo disco
```
fdisk /dev/vdc
g  -> cria uma nova tabela de partição GPT vazia
n  -> cria uma nova partição
 1 -> primeira partição
 2048 -> primeiro setor da partição
 +50M -> partição terá 50Mb


 n -> cria uma nova partição
 2 -> primeira partição
 104448   -> primeiro setor da partição (logo após os 50Mb)
 12582911 -> partição usará o resto do disco

 t -> altera o tipo da partição
 2 -> alterar a partição 1
 1 -> EFI - ESP

 w -> grava a tabela no disco e sai
```

## Vendo como ficou a tabela no disco
```
fdisk -l /dev/vdc

Disk /dev/vdc: 6 GiB, 6442450944 bytes, 12582912 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 6C3A8B10-C9FD-5749-9DA1-75069DD8EE22

Device      Start      End  Sectors Size Type
/dev/vdc1    2048   104447   102400  50M EFI System
/dev/vdc2  104448 12582878 12478431   6G Linux filesystem
```

## Criando Luks

### Instalando ferramenta necessária
```
apt install cryptsetup
```

### Preparando a partição
```
cryptsetup luksFormat --type luks1 /dev/vdc2

WARNING!
========
This will overwrite data on /dev/vdc2 irrevocably.

Are you sure? (Type uppercase yes): YES
Enter passphrase for /dev/vdc2:
Verify passphrase:
```

### Vendo como ficou
```
cryptsetup luksDump /dev/vdc2
LUKS header information for /dev/vdc2

Version:       	1
Cipher name:   	aes
Cipher mode:   	xts-plain64
Hash spec:     	sha256
Payload offset:	4096
MK bits:       	512
MK digest:     	7b d8 70 54 d7 c6 cd 4f 6d d0 94 d5 b2 76 88 d4 c3 1f 15 30
MK salt:       	6f a9 9e 2e 8c 82 95 24 19 42 bc 53 67 da c2 6c
               	aa 83 96 b1 60 d4 cc 27 c8 32 5a 62 cf 43 05 43
MK iterations: 	72817
UUID:          	957fc31f-0df0-47e5-bd19-6d6a7338a156

Key Slot 0: ENABLED
	Iterations:         	1159928
	Salt:               	75 a6 76 1f 41 20 7b a0 99 d9 23 0f c2 43 c4 2e
	                      	35 80 ef b5 f6 0f 1b f9 9d d6 06 60 86 c0 da 2a
	Key material offset:	8
	AF stripes:            	4000
Key Slot 1: DISABLED
Key Slot 2: DISABLED
Key Slot 3: DISABLED
Key Slot 4: DISABLED
Key Slot 5: DISABLED
Key Slot 6: DISABLED
Key Slot 7: DISABLED
```

### Criando um mapeamento para partição luks
```
cryptsetup luksOpen /dev/vdc2 criptografado
```
Digite a senha para /dev/vdc2: -> mesma senha usada antes

### Mapeamento: /dev/mapper/criptografado
```
lsblk

vdc               252:32   0    6G  0 disk
├─vdc1            252:33   0   50M  0 part
└─vdc2            252:34   0    6G  0 part
  └─criptografado 253:1    0    6G  0 crypt
```

### Criando sistemas de arquivos ("formatando")
```
mkfs.ext4 /dev/mapper/criptografado
mkfs.vfat /dev/vdc1
```

### Criando os pontos de montagem
```
mkdir /antigo
mkdir /novo/
```

### Montando origem e destino
```
mount /dev/vda2 /antigo
mount /dev/vda1 /antigo/boot/efi

mount /dev/mapper/criptografado /novo
mkdir -p /novo/boot/efi
mount /dev/vdc1 /novo/boot/efi

lsblk

NAME              MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
vda               252:0    0   20G  0 disk
├─vda1            252:1    0   50M  0 part  /antigo/boot/efi
└─vda2            252:2    0   20G  0 part  /antigo
vdc               252:32   0    6G  0 disk
├─vdc1            252:33   0   50M  0 part  /novo/boot/efi
└─vdc2            252:34   0    6G  0 part
  └─criptografado 253:1    0    6G  0 crypt /novo

mount
...

/dev/vda2 on /antigo type ext4 (rw,relatime)
/dev/vda1 on /antigo/boot/efi type vfat (rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro)
/dev/mapper/criptografado on /novo type ext4 (rw,relatime)
/dev/vdc1 on /novo/boot/efi type vfat (rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro)
```

### Preparando par sincronizar
Instalando rsync
```
apt install rsync
```

### Sincronozando os arquivos
Sincronizando...
```
rsync -av /antigo/* /novo/ --exclude=/antigo --exclude=/novo
```
Se for uma **pessoa desconfiada**, roda de novo, desta vez demora quase nada.
```
rsync -av /antigo/* /novo/ --exclude=/antigo --exclude=/novo
```

### Desmontando o /antigo
```
umount /antigo/boot/efi
umount /antigo
```

### Preparando para chrootar
```
cd /novo
mount --bind /proc proc
mount --bind /dev dev
mount --bind /sys sys
```
### Chrootando
```
chroot .
```
obs. agora é no sistema novo!

### Preparando initrd para poder abrir luks
```
cp /usr/share/initramfs-tools/hooks/cryptkeyctl /etc/initramfs-tools/hooks
```

### Analizando UUIDs
```
ls -l /dev/disk/by-uuid/
total 0
lrwxrwxrwx 1 root root 10 mar  8 18:55 0752681a-deb7-48d4-80ab-856f960ce321 -> ../../dm-0
lrwxrwxrwx 1 root root 10 mar  8 13:09 1029-1F02 -> ../../vda1
lrwxrwxrwx 1 root root 10 mar  8 13:09 336dbd2e-3e4f-4fba-9a01-6ea17e8b1802 -> ../../vda2
lrwxrwxrwx 1 root root 10 mar  8 18:52 957fc31f-0df0-47e5-bd19-6d6a7338a156 -> ../../vdc2
lrwxrwxrwx 1 root root 10 mar  8 18:55 9B75-9646 -> ../../vdc1
```

### Editando arquivo de configuração /etc/crypttab
```
cat /etc/crypttab

# <target name>	<source device>		<key file>	<options>
criptografado UUID=957fc31f-0df0-47e5-bd19-6d6a7338a156 none luks,discard
```

### Preparando novo fstab
```
apt install arch-install-scripts
genfstab  / >/etc/fstab
```
### Vendo como ficou o fstab
```
cat /etc/fstab

# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
/dev/mapper/criptografado	/         	ext4      	rw,relatime	0 1

/dev/vdc1           	/boot/efi 	vfat      	rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro	0 2
```

### Editando grub (/etc/default/grub)
```
GRUB_CMDLINE_LINUX="root=/dev/mapper/criptografado cryptdevice=UUID=957fc31f-0df0-47e5-bd19-6d6a7338a156:criptografado"
GRUB_ENABLE_CRYPTODISK=y
```

### Instalando GRUB no disco novo
```
grub-install /dev/vdc

Installing for x86_64-efi platform.
Installation finished. No error reported.
```

### Criando uma chave
** Para não ter que digitar a senha 2x, uma para o grub outra para o kernel**
```
dd bs=512 count=4 if=/dev/urandom of=/crypto_keyfile.bin
chmod 000 /crypto_keyfile.bin
```

### Adicionando uma chave no slot
```
cryptsetup luksAddKey /dev/vdc2 /crypto_keyfile.bin
```
Digite qualquer senha existente:

**Para que nenhum usuário local sem ser root possa ler o initrd e copiar a chave**\
**incluir no arquivo /etc/initramfs-tools/initramfs.conf**
```
UMASK=0077
```

### Para incluir chave no initramfs
  - no arquivo /etc/cryptsetup-initramfs/conf-hook
  - definir as variáveis dessa forma:
```
CRYPTSETUP=y
KEYFILE_PATTERN=/*.bin
```

### Editando arquivo de configuração /etc/crypttab
```
cat /etc/crypttab
# <target name> <source device>         <key file>      <options>
criptografado UUID=957fc31f-0df0-47e5-bd19-6d6a7338a156 /crypto_keyfile.bin luks,discard
```

### Agora atualizamos o initramfs
```
update-initramfs -u -k all

update-initramfs: Generating /boot/initrd.img-5.5.8-gnu
```

### Criando arquivo de configuração do GRUB
Para não executar os_prober
```
chmod -x /etc/grub.d/30_os-prober

update-grub
```

### Saindo do chroot e desmontando tudo
```
exit

cd
umount /novo/proc
umount /novo/dev
umount /novo/sys/fs/fuse/connections
umount /novo/sys/
umount /novo/boot/efi
umount /novo

cryptsetup luksClose criptografado

```

### Reiniciar o sistema
```
shutdown -r now
```

### Trocar a ordem dos dispositivos de boot na BIOS

### Caso tenha problemas (troubleshooting)
Caso seu sistema não dê boot, pode usar o console do GRUB para dar boot e resolver o problema.\
Se não conseguir ver a tela do Grub, pode usar de um live-cd do Debian.

Na tela do GRUB tecle *c* para usar o shell do Grub.

rode: `ls` para ver os nomes dos discos.\
Você vai precisar saber qual é o do seu sistema.\
Para montar decriptografando, use os comandos abaixo, será necessário digitar a senha que definiu para o luks.
```
insmod luks
cryptomount (hd1,gpt2)
configfile (crypto0)/boot/grub/grub.cfg
```
Agora na tela do grub do seu sistema novo.
Dê boot, logue como root, para verificar o que pode ter dado errado.

- Algumas coisas a verificar:
```
lsblk
cat /etc/crypttab
```

- Compare se está correto o UUID ou o nome do dispositivo que usou.

- Um erro comum é não ter incluído a entrada na UEFI.
Para verificar:
```
efibootmgr -v
```
Caso não esteja lá rode:
```
update-grub
```

- Verifique o /etc/fstab.
- Verifique se o nome da chave está correto no arquivo /etc/crypttab: `/crypto_keyfile.bin`
- Verifique se está no slot do luks.
```
cryptsetup luksDump /dev/vdc2
```

- Verifique o arquivo /boot/efi/EFI/debian/grub.cfg
Veja se os UUIDs estão corretos, se não estiver mova o arquivo para outro diretório e rode:
```
grub-install /dev/vdc
update-grub
```

### Conclusão
A "nova" instalação está agora mais protegida, mesmo que alguém tenha acesso físico a sua máquina, não conseguirá ter acesso ao seus dados, nem mesmo rodar seu sistema.

No entando ainda é vulnerável ao "Ataque da diabólica camareira"\
[to Evil Maid attacks.](https://www.schneier.com/blog/archives/2009/10/evil_maid_attac.html)

