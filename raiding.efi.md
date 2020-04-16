# Transformando uma instalação, com raid1 numa máquina com UEFI
Temos uma instalação de Debian em um disco, queremos colocar um segundo disco do mesmo tamanho e colocá-los em raid1 (espelhamento).

1. Preparando ambiente
2. Criando a tabela de partições
3. Criando o raid1
4. Copiando os arquivos
5. Preparando boot
6. Boot no sistema com raid
7. Incluindo disco1 ao raid.
8. Preparando boot
9. Boot

## Etapa 1 (Preparando ambiente)

Vamos fazer esse tutorial numa máquina virtual usando kvm/qemu, mas o procedimento será o mesmo numa máquina real.

Se estiver fazendo numa máquina real pode ir para etapa 2.

A máquina virtual (VM) que vou usar tem um disco de 7G, portanto vou criar um outro disco com o mesmo tamanho.
```
qemu-img create -f qcow2 raid1.qcow2 7G
```
Coloquei o disco raid1.qcow2 na VM e dei boot no sistema.

## Etapa 2 (Criando a tabela de partições)

Primeiro vou verificar se os discos são do mesmo tamanho.
```
fdisk -l

Disco /dev/vda: 7 GiB, 7516192768 bytes, 14680064 setores
Unidades: setor de 1 * 512 = 512 bytes
Tamanho de setor (lógico/físico): 512 bytes / 512 bytes
Tamanho E/S (mínimo/ótimo): 512 bytes / 512 bytes
Tipo de rótulo do disco: dos
Identificador do disco: 0x29e236dd

Dispositivo Inicializar Início      Fim  Setores Tamanho Id Tipo
/dev/vda1   *             2048 14680063 14678016      7G 83 Linux

Disco /dev/vdb: 7 GiB, 7516192768 bytes, 14680064 setores
Unidades: setor de 1 * 512 = 512 bytes
Tamanho de setor (lógico/físico): 512 bytes / 512 bytes
Tamanho E/S (mínimo/ótimo): 512 bytes / 512 bytes
```
Repare que o segundo disco nem tem um particionamento, no seu caso pode ser que o segundo disco já tenha um particionamento, não precisa se preocupar com isso.

Vamos criar no segundo disco o particionamento que desejamos para o raid1.\
Aqui você pode aproveitar para reformular o particionamento do sistema instalado.

Vamos usar o programa *sgdisk* do pacote gdisk, caso não o tenha instalado rode:
```
apt install gdisk
```

Copiando a tabela de partição do primeiro para o segundo disco.
```
sfdisk -d /dev/vda | sfdisk --force /dev/vdb

Checking that no-one is using this disk right now ... OK

Disk /dev/vdb: 7 GiB, 7516192768 bytes, 14680064 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Created a new GPT disklabel (GUID: C410FA1C-17C3-4740-95BD-8780D92C49B6).
/dev/vdb1: Created a new partition 1 of type 'EFI System' and of size 50 MiB.
/dev/vdb2: Created a new partition 2 of type 'Linux filesystem' and of size 7 GiB.
/dev/vdb3: Done.

New situation:
Disklabel type: gpt
Disk identifier: C410FA1C-17C3-4740-95BD-8780D92C49B6

Device      Start      End  Sectors Size Type
/dev/vdb1    2048   104447   102400  50M EFI System
/dev/vdb2  104448 14680030 14575583   7G Linux filesystem

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

Vendo como ficou:
```
fdisk -l /dev/vdb

Disk /dev/vdb: 7 GiB, 7516192768 bytes, 14680064 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: C410FA1C-17C3-4740-95BD-8780D92C49B6

Device      Start      End  Sectors Size Type
/dev/vdb1    2048   104447   102400  50M EFI System
/dev/vdb2  104448 14680030 14575583   7G Linux filesystem
```

## Etapa 3 (Criando os raids)
Para criar o raid vamos usar *mdadm* caso não tenha instalado rode:
```
apt install mdadm
```

Como pretendemos preservar os dados do primeiro disco, para depois copiar para o raid, vamos usar uma estratégia de criar o raid1 de dois discos usando nesse momento apenas o segundo disco. 

```
mdadm --create /dev/md1 --level=1 --metadata 1.0 --raid-devices=2 missing /dev/vdb1
mdadm --create /dev/md2 --level=1 --metadata 1.0 --raid-devices=2 missing /dev/vdb2
```
Usaremos o GRUB de bootloader que é capaz de entender os metadados md/v1.0.

### Vendo como ficou

```
cat /proc/mdstat

md2 : active raid1 vdb2[1]
      7282624 blocks super 1.2 [2/1] [_U]

md1 : active raid1 vdb1[1]
      50176 blocks super 1.2 [2/1] [_U]

```
Repare na indicação `[_U]` isso mostra que o primeiro disco não está presente no raid1 nesse momento.

Vendo como estão os UUIDs dos dispositos
```
blkid

/dev/vda1: SEC_TYPE="msdos" UUID="413B-DC20" TYPE="vfat" PARTUUID="acb69e58-1a84-4705-92b0-55d5616d6ca6"
/dev/vda2: UUID="002e0491-5917-42eb-86d0-4e9426d7d265" TYPE="ext4" PARTUUID="41b7434a-089a-4031-9da4-984d13ba121f"
/dev/vdb1: UUID="5295c340-da42-6c47-fc43-0edfe7b50db6" UUID_SUB="5bae51d2-f46f-cc08-cd1c-bd007725114b" TYPE="linux_raid_member" PARTUUID="acb69e58-1a84-4705-92b0-55d5616d6ca6"
/dev/vdb2: UUID="d98327b0-992f-503b-9182-9e3cbb365e86" UUID_SUB="77c15ade-fa1d-2cf3-6ea2-5dc4b79cdced" TYPE="linux_raid_member" PARTUUID="41b7434a-089a-4031-9da4-984d13ba121f"

```
## Etapa 4 (Copiando os arquivos)

### Criar os sistemas de arquivos (fs) e montar o raid
Com sistema de arquivos FAT para o UEFI.
```
mkfs.vfat -F32 /dev/md1
mkfs.fat 4.1 (2017-01-24)
```

Com sistema de arquivos ext4 para o sistema.
```
mkfs.ext4 /dev/md2

mke2fs 1.44.5 (15-Dec-2018)
Creating filesystem with 1833472 4k blocks and 458752 inodes
Filesystem UUID: d155b3b0-82a1-480e-8800-742897a917b1
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```

### Montando o raid

```
mkdir /novo
mount /dev/md2 /novo
mkdir -p /novo/boot/efi
mount /dev/md1 /novo/boot/efi
```

### Montar as partições do sistema em outro ponto de montagem
```
mkdir /antigo
mount /dev/vda2 /antigo
mount /dev/vda1 /antigo/boot/efi
```

### Observando os dispositivos
```
lsblk

NAME    MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sr0      11:0    1  2,3G  0 rom
vda     254:0    0    7G  0 disk
├─vda1  254:1    0   50M  0 part  /antigo/boot/efi
└─vda2  254:2    0    7G  0 part  /antigo
vdb     254:16   0    7G  0 disk
├─vdb1  254:17   0   50M  0 part
│ └─md1   9:1    0   50M  0 raid1 /novo/boot/efi
└─vdb2  254:18   0    7G  0 part
  └─md2   9:2    0    7G  0 raid1 /novo
```

### Copiar os dados do hd antigo para o raid1 novo
Para copiar os arquivos vamos usar *rsync* caso não tenha instalado rode:
```
apt install rsync
```

### Copiando o conteúdo para o raid
```
rsync -av --update /antigo/* /novo
```

### Craindo um fstab
Vamos usar o *genfstab* que está no pacote *arch-install-scripts*, caso não tenha instaado rode:
```
apt install arch-install-scripts
```

### Criando fstab
```
genfstab -U /novo > /novo/etc/fstab
```
Verificando:
```
cat /novo/etc/fstab

# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# /dev/md2
UUID=69ba4755-b8ca-45de-93ad-97dbde3912f2	/         	ext4      	rw,relatime	0 1

# /dev/md1
UUID=DCD7-56C6      	/boot/efi 	vfat      	rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro	0 2
```
Vendo os blocos.
```
lsblk

NAME    MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
sr0      11:0    1 1024M  0 rom
vda     254:0    0    7G  0 disk
├─vda1  254:1    0   50M  0 part  /antigo/boot/efi
└─vda2  254:2    0    7G  0 part  /antigo
vdb     254:16   0    7G  0 disk
├─vdb1  254:17   0   50M  0 part
│ └─md1   9:1    0   49M  0 raid1 /novo/boot/efi
└─vdb2  254:18   0    7G  0 part
  └─md2   9:2    0    7G  0 raid1 /novo
```
Vendo os IDs das partições
```
blkid

/dev/vda1: SEC_TYPE="msdos" UUID="413B-DC20" TYPE="vfat" PARTUUID="acb69e58-1a84-4705-92b0-55d5616d6ca6"
/dev/vda2: UUID="002e0491-5917-42eb-86d0-4e9426d7d265" TYPE="ext4" PARTUUID="41b7434a-089a-4031-9da4-984d13ba121f"
/dev/vdb1: UUID="ec325838-3d74-4fce-821f-f1f645c053f0" UUID_SUB="4842f05a-1af6-da02-5661-10e0fb431865" LABEL="pureos:1" TYPE="linux_raid_member" PARTUUID="acb69e58-1a84-4705-92b0-55d5616d6ca6"
/dev/vdb2: UUID="175a8cd1-7274-169e-5273-85b5fe9eb4f6" UUID_SUB="099732fb-1082-3b96-c2d3-3d60db84d7a8" LABEL="pureos:2" TYPE="linux_raid_member" PARTUUID="41b7434a-089a-4031-9da4-984d13ba121f"
/dev/md1: UUID="DCD7-56C6" TYPE="vfat"
/dev/md2: UUID="69ba4755-b8ca-45de-93ad-97dbde3912f2" TYPE="ext4"
```

## Etapa 5 (Preparando boot)

### chroot no novo
```
cd /novo

mount --bind /sys sys
mount --bind /proc proc
mount --bind /dev dev

chroot .
```

### Criando o arquivo de configuração do raid
```
mdadm --detail --scan >> /etc/mdadm.conf
```
Verificando:
```
cat /etc/mdadm.conf
ARRAY /dev/md1 metadata=1.0 name=pureos:1 UUID=ec325838:3d744fce:821ff1f6:45c053f0
ARRAY /dev/md2 metadata=1.2 name=pureos:2 UUID=175a8cd1:7274169e:527385b5:fe9eb4f6
```

### Removendo os diretórios que serviram de ponto de montagem
**Verifique se está no chroot mesmo** rodando:
```
ls /novo
ls /antigo
```
Em ambos os casos os diretórios devem estar vazios, só então pode remover:
```
rm /novo /antigo -r
```

### Atualizando o initrd
```
update-initramfs -u -k all
```

### Recriando o arquivo de configuração do grub
```
update-grub

/usr/sbin/grub-probe: aviso: Não foi possível encontrar o volume físico '(null)'. Alguns módulos poderão faltar na imagem core..
```
Esses alertas ocorrem porque o raid está num estado chamado "degradado", pois ainda tem apenas um disco.

### Saindo do chroot e desmontando tudo.
```
exit

umount sys
umount proc
umount dev
cd
umount /novo/boot/efi
umount /novo

umount /antigo/boot/efi
umount /antigo
```

### Rebootando...
```
shutdown -r now
```
Sugiro fortemente que tenha um live-cd ou um pendrive com o Grub, no caso de algo dar errado e precisar de um grub para carregar o sistema e corrigir.

Dando tudo certo...

O sistema vai mostrar o GRUB com as configurações da instalação antiga, vamos usar o console do grub para dar boot na instalação que já está no raid.

## Etapa 6 (Boot no sistema com raid)

### Boot novo hd
Na tela do grub tecle c para ir para o console do grub.
```
ls

(hd0) (hd0,gpt2) (hd0,gpt1) (hd1) (hd1,gpt2) (hd1,gpt1)

```
Isso mostrará os discos que tem. Se rodar `ls (hd0,gpt2)/` verá o conteúdo da partição que está fora do raid.\
Se rodar `ls (hd1,gpt2)/` deve obter uma mensagem de erro, pois o grub ainda não é capaz de entender o raid.
Vamos carregar o módulo do grub que dá essa capacidade:
```
insmod mdraid1x
```
Agora rodando `ls` verá mais dispositivos, porque agora se vê os raids.
```
ls
(proc) (hd0) (hd0,gpt2) (hd0,gpt1) (hd1) (hd1,gpt2) (hd1,gpt1) (md/2) (md/1)
```
Vamos carregar o arquivo de configuração do grub que está no disco em raid e dar boot por ele.
```
configfile (md/2)/boot/grub/grub.cfg
```
Você verá a tela do grub, já com as configurações do raid, caso queira conferir tecle e.

## Etapa 7 (Incluindo disco1 aos raids)

Logue no sistema para continuar.

### Checagem de pontos de montagem
```
lsblk

NAME    MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
vda     254:0    0    7G  0 disk
├─vda1  254:1    0   50M  0 part
└─vda2  254:2    0    7G  0 part
vdb     254:16   0    7G  0 disk
├─vdb1  254:17   0   50M  0 part
│ └─md1   9:1    0   49M  0 raid1 /boot/efi
└─vdb2  254:18   0    7G  0 part
  └─md2   9:2    0    7G  0 raid1 /
```
Repare que o / está montado no raid1 e que as partições vda1 e vda2 não estão desmontadas.

### incluir as partições do primeiro disco aos raids
```
mdadm /dev/md1 -a /dev/vda1
mdadm /dev/md2 -a /dev/vda2
```

### Verificando o progresso da sincronização do array
```
cat /proc/mdstat

Personalities : [raid1] [linear] [multipath] [raid0] [raid6] [raid5] [raid4] [raid10]
md1 : active raid1 vda1[2] vdb1[1]
      50176 blocks super 1.2 [2/2] [UU]

md2 : active raid1 vda2[2] vdb2[1]
      7282624 blocks super 1.2 [2/1] [_U]
      [==>..................]  recovery = 12.5% (910336/7282624) finish=0.5min speed=182067K/sec

unused devices: <none>
```

Rode o comando mais de uma vez até que termine a sincronização, verá assim:
```
cat /proc/mdstat

Personalities : [raid1] [linear] [multipath] [raid0] [raid6] [raid5] [raid4] [raid10]
md1 : active raid1 vda1[2] vdb1[1]
      50176 blocks super 1.2 [2/2] [UU]

md2 : active raid1 vda2[2] vdb2[1]
      7282624 blocks super 1.2 [2/2] [UU]

unused devices: <none>
```
Raids ok, veja que agora mostra `[UU]` evidenciando a integridade do raid.

### Veja os dispositivos agora
```
lsblk

NAME    MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
vda     254:0    0    7G  0 disk
├─vda1  254:1    0   50M  0 part
│ └─md1   9:1    0   49M  0 raid1 /boot/efi
└─vda2  254:2    0    7G  0 part
  └─md2   9:2    0    7G  0 raid1 /
vdb     254:16   0    7G  0 disk
├─vdb1  254:17   0   50M  0 part
│ └─md1   9:1    0   49M  0 raid1 /boot/efi
└─vdb2  254:18   0    7G  0 part
  └─md2   9:2    0    7G  0 raid1 /
```

## Etapa 8 (Preparando boot)

### Ajustando o grub
```
update-grub
```

### Gravando a entrada UEFI para cada um dos discos
```
efibootmgr -c -d /dev/vda -p 1 -L "Debian1" -l "\EFI\debian\grubx64.efi"
efibootmgr -c -d /dev/vdb -p 1 -L "Debian2" -l "\EFI\debian\grubx64.efi"
```

## Etapa 9

### Rebootando...
```
shutdown -r now
```
Sugiro novamente que tenha um live-cd ou um pendrive com o Grub, no caso de algo dar errado e precisar de um grub para carregar o sistema e corrigir.

Dando tudo certo...

O sistema vai mostrar o GRUB com as configurações novas e poderá dar boot na instalação que já está no raid.

### Conclusão
Com esse tutorial é possível ter sua instalação agora usando raid1 (espelhamento) e caso algum dos discos apresente problemas o sistema continuará funcionando.

Qualquer dúvida me procure no Telegram!

- [Grupo Debian Brasil](https://t.me/debianbrasil)
- [Grupo Debian BR](https://t.me/debianbr)

