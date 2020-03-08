# Transplantando instalação e implementando Lucks

## A máquina tem 2 HDs
```
fdisk -l

Disk /dev/vda: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2EA7B5C3-1854-0C47-950B-C48585611FC1

Device        Start      End  Sectors Size Type
/dev/vda1      2048   104447   102400  50M EFI System
/dev/vda2    104448 10590207 10485760   5G Linux swap
/dev/vda3  10590208 41943006 31352799  15G Linux filesystem


Disk /dev/vdb: 6 GiB, 6442450944 bytes, 12582912 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x34960aab
```

## Preparando o segundo disco
```
fdisk /dev/vdb
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
fdisk -l /dev/vdb
Disk /dev/vdb: 6 GiB, 6442450944 bytes, 12582912 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: DAEA47DE-0D58-704E-8347-27E93CD9DE59

Device      Start      End  Sectors Size Type
/dev/vdb1    2048   104447   102400  50M EFI System
/dev/vdb2  104448 12582878 12478431   6G Linux filesystem
```

## Criando Luks

### Instalando ferramenta necessária
```
apt install cryptsetup
```

### Preparando a partição
```
cryptsetup luksFormat --type luks1 /dev/vdb2

**WARNING!**
Isto vai sobrescrever dados em /dev/vdb2 permanentemente.

Are you sure? (Type uppercase yes): YES
Digite a senha para /dev/vdb2:
Verificar senha:
```

### Vendo como ficou
```
cryptsetup luksDump /dev/vdb2

LUKS header information
Version:       	1
Epoch:         	3
Metadata area: 	16384 [bytes]
Keyslots area: 	16744448 [bytes]
UUID:          	5c4ab8fc-76dc-44c5-84b5-a5ca4dff9bb6
Label:         	(no label)
Subsystem:     	(no subsystem)
Flags:       	(no flags)

Data segments:
  0: crypt
	offset: 16777216 [bytes]
	length: (whole device)
	cipher: aes-xts-plain64
	sector: 512 [bytes]

Keyslots:
  0: luks1
	Key:        512 bits
	Priority:   normal
	Cipher:     aes-xts-plain64
	Cipher key: 512 bits
	PBKDF:      argon2i
	Time cost:  4
	Memory:     303407
	Threads:    1
	Salt:       5c 0b 42 2f e8 a2 69 19 c4 44 e7 8f e0 91 0b 59
	            09 52 da f0 49 c1 b1 71 c9 ff df 98 ef b5 be 72
	AF stripes: 4000
	AF hash:    sha256
	Area offset:32768 [bytes]
	Area length:258048 [bytes]
	Digest ID:  0
Tokens:
Digests:
  0: pbkdf1
	Hash:       sha256
	Iterations: 66805
	Salt:       a5 0f fd 0d 03 3a a3 c6 40 db 24 72 d6 6c 39 29
	            2a 77 7b 49 87 03 aa a7 2d 8f 97 26 bc af 2a d6
	Digest:     0b 6f 98 ae f3 b4 74 00 4c fe 7a fc 61 d4 a9 13
	            9f a8 d9 dc 8c 49 a5 53 7d 65 83 4a e5 8e 1e 2b

```
### Criando um mapeamento para partição luks
```
cryptsetup luksOpen /dev/vdb2 criptografado
```
Digite a senha para /dev/vdb2: -> mesma senha usada antes

### Mapeamento: /dev/mapper/criptografado
```
lsblk

vdb               252:16   0    6G  0 disk
├─vdb1            252:17   0   50M  0 part
└─vdb2            252:18   0    6G  0 part
  └─criptografado 253:0    0    6G  0 crypt
```

### Criando sistemas de arquivos ("formatando")
```
mkfs.ext4 /dev/mapper/criptografado
mkfs.vfat /dev/vdb1
```

### Criando os pontos de montagem
```
mkdir /antigo
mkdir /novo/
```

### Montando origem e destino
```
mount /dev/vda3 /antigo
mount /dev/vda1 /antigo/boot/efi

mount /dev/mapper/criptografado /novo
mkdir -p /novo/boot/efi
mount /dev/vdb1 /novo/boot/efi

lsblk

vda               252:0    0   20G  0 disk
├─vda1            252:1    0   50M  0 part  /antigo/boot/efi
├─vda2            252:2    0    5G  0 part  [SWAP]
└─vda3            252:3    0   15G  0 part  /antigo
vdb               252:16   0    6G  0 disk
├─vdb1            252:17   0   50M  0 part  /novo/boot/efi
└─vdb2            252:18   0    6G  0 part
  └─criptografado 253:0    0    6G  0 crypt /novo


mount

/dev/vda3 on /antigo type ext4 (rw,relatime)
/dev/mapper/criptografado on /novo type ext4 (rw,relatime)
/dev/vdb1 on /novo/boot/efi type vfat (rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro)
/dev/vda1 on /antigo/boot/efi type vfat (rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro)
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
lrwxrwxrwx 1 root root 10 fev 19 23:17 1029-1F02 -> ../../vda1
lrwxrwxrwx 1 root root 10 fev 19 23:40 1ACA-FCE4 -> ../../vdb1
lrwxrwxrwx 1 root root 10 fev 19 23:17 336dbd2e-3e4f-4fba-9a01-6ea17e8b1802 -> ../../vda3
lrwxrwxrwx 1 root root 10 fev 19 23:39 5c4ab8fc-76dc-44c5-84b5-a5ca4dff9bb6 -> ../../vdb2
lrwxrwxrwx 1 root root 10 fev 19 23:17 61e56665-c214-4fb4-a6dd-0cf1d62473ee -> ../../vda2
lrwxrwxrwx 1 root root 10 fev 19 23:40 f2f8c55e-bdd5-41cd-849b-343533b3a717 -> ../../dm-0
```

### Editando arquivo de configuração /etc/crypttab
```
cat /etc/crypttab

# <target name>	<source device>		<key file>	<options>
criptografado UUID=5c4ab8fc-76dc-44c5-84b5-a5ca4dff9bb6 none luks,discard
```

### Preparando novo fstab
```
apt install arch-install-scripts
genfstab  / >/etc/fstab
```
Rremovi o swap
```
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust Way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>   <dump>  <pass>
#
dev/mapper/criptografado       /               ext4            rw,relatime     0 1
/dev/vdb1               /boot/efi       vfat            rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro    0 2
```

### Editando grub (/etc/default/grub)
```
GRUB_CMDLINE_LINUX="root=/dev/mapper/criptografado cryptdevice=UUID=5c4ab8fc-76dc-44c5-84b5-a5ca4dff9bb6:criptografado"
GRUB_ENABLE_CRYPTODISK=y
```

### Instalando GRUB no disco novo
```
grub-install /dev/vdb
```

### Atualizando o initramfs
```
update-initramfs -u -k all
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

** Para não ter que digitar a senha 2x, uma para o grub outra para o kernel**

### Criando uma chave
```
dd bs=512 count=4 if=/dev/urandom of=/crypto_keyfile.bin
chmod 000 /crypto_keyfile.bin
chmod 600 /boot/initrd.img*
```

### Adicionando uma chave no slot
```
cryptsetup luksAddKey /dev/vdb2 /crypto_keyfile.bin
```
Digite qualquer senha existente:

**Para que nenhum usuário local sem ser root possa ler o initrd e copiar a chave**
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
criptografado UUID=0a1c1f47-aafd-472c-8a00-82c58a63e55b /crypto_keyfile.bin luks,discard
```

### Agora atualizamos o initramfs
```
update-initramfs -u -k all
```

### Reboot
```
shutdown -r now
```



