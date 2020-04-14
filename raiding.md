# Transformando uma instalação, com raid1
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
Nesse exemplo usamos uma máquina que tem BIOS e para o Grub poder funcionar vamos criar uma pequena partição de 1M no inicio do disco.

```
fdisk /dev/vdb

Bem-vindo ao fdisk (util-linux 2.33.1).
As alterações permanecerão apenas na memória, até que você decida gravá-las.
Tenha cuidado antes de usar o comando de gravação.

Comando (m para ajuda): o
Criado um novo rótulo de disco DOS com o identificador de disco 0x038b3193.

Comando (m para ajuda): n
Tipo da partição
   p   primária (0 primárias, 0 estendidas, 4 livre)
   e   estendida (recipiente para partições lógicas)
Selecione (padrão p):

Usando resposta padrão p.
Número da partição (1-4, padrão 1):
Primeiro setor (2048-14680063, padrão 2048):
Último setor, +/-setores ou +/-tamanho{K,M,G,T,P} (2048-14680063, padrão 14680063): +1M

Criada uma nova partição 1 do tipo "Linux" e de tamanho 1 MiB.

Comando (m para ajuda): t
Selecionou a partição 1
Código hexadecimal (digite L para listar todos os códigos): da
O tipo da partição "Linux" foi alterado para "Non-FS data".

Comando (m para ajuda): n
Tipo da partição
   p   primária (1 primárias, 0 estendidas, 3 livre)
   e   estendida (recipiente para partições lógicas)
Selecione (padrão p):

Usando resposta padrão p.
Número da partição (2-4, padrão 2):
Primeiro setor (4096-14680063, padrão 4096):
Último setor, +/-setores ou +/-tamanho{K,M,G,T,P} (4096-14680063, padrão 14680063):

Criada uma nova partição 2 do tipo "Linux" e de tamanho 7 GiB.

Comando (m para ajuda): w
A tabela de partição foi alterada.
Chamando ioctl() para reler tabela de partição.
Sincronizando discos.
```

Vendo como ficou:
```
fdisk -l /dev/vdb
Disco /dev/vdb: 7 GiB, 7516192768 bytes, 14680064 setores
Unidades: setor de 1 * 512 = 512 bytes
Tamanho de setor (lógico/físico): 512 bytes / 512 bytes
Tamanho E/S (mínimo/ótimo): 512 bytes / 512 bytes
Tipo de rótulo do disco: dos
Identificador do disco: 0xdc578e40

Dispositivo Inicializar Início      Fim  Setores Tamanho Id Tipo
/dev/vdb1                 2048     4095     2048      1M da Dados Não-FS
/dev/vdb2                 4096 14680063 14675968      7G 83 Linux
```

## Etapa 3 (Criando o raid1)
Para criar o raid vamos usar *mdadm* caso não tenha instalado rode:
```
apt install mdadm
```

Como pretendemos preservar os dados do primeiro disco, para depois copiar para o raid, vamos usar uma estratégia de criar o raid1 de dois discos usando nesse momento apenas o segundo disco. 

```
mdadm --create /dev/md1 --level=1 --raid-devices=2 missing /dev/vdb2

mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
```
Usaremos o GRUB de bootloader que é capaz de entender os metadados md/v1.x não devemos nos preocupar com esse alerta.

### Vendo como ficou

```
cat /proc/mdstat

cat /proc/mdstat
Personalities : [raid1]
md1 : active raid1 vdb2[1]
      7332864 blocks super 1.2 [2/1] [_U]

unused devices: <none>

```
Repare na indicação `[_U]` isso mostra que o primeiro disco não está presente no raid1 nesse momento.

Vendo como estão os UUIDs dos dispositos
```
blkid

/dev/vda1: UUID="3fdcfed0-225c-4bdc-8e38-59072bf7aa6a" TYPE="ext4" PTTYPE="dos" PARTUUID="29e236dd-01"
/dev/vdb1: PARTUUID="dc578e40-01"
/dev/vdb2: UUID="c50a66b1-e697-2fc0-6bec-42f3a4638eb1" UUID_SUB="3f0f1a2c-64e7-bec7-2f44-c881b2e0632c" LABEL="debian:1" TYPE="linux_raid_member" PARTUUID="dc578e40-02"
```
## Etapa 4 (Copiando os arquivos)

### Criar os sistemas de arquivos (fs) e montar o raid
```
mkfs.ext4 /dev/md1

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
mount /dev/md1 /novo
```

### Montar as partições do sistema em outro ponto de montagem
```
mkdir /antigo
mount /dev/vda1 /antigo
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
# /dev/md1
UUID=a0e668eb-21be-4254-ba80-93f1375858c6	/         	ext4      	rw,relatime	0 1
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
ARRAY /dev/md1 metadata=1.2 name=debian:1 UUID=c50a66b1:e6972fc0:6bec42f3:a4638eb1
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
### Gravando grub no raid
```
grub-install /dev/md1
Instalando para a plataforma i386-pc.
grub-install: aviso: Não foi possível encontrar o volume físico '(null)'. Alguns módulos poderão faltar na imagem core..
grub-install: aviso: Não foi possível encontrar o volume físico '(null)'. Alguns módulos poderão faltar na imagem core..
grub-install: aviso: File system `ext2' doesn't support embedding.
grub-install: aviso: Embedding is not possible.  GRUB can only be installed in this setup by using blocklists.  However, blocklists are UNRELIABLE and their use is discouraged..
grub-install: erro: will not proceed with blocklists.
```
Esses alertas ocorrem porque o raid está num estado chamado "degradado", pois ainda tem apenas um disco.
O erro ocorre porque o grub nâo encontra espaço no raid, resolveremos isso adiante.

### Atualizando o initrd
```
update-initramfs -u -k all
```

### Instalando o grub e recriando o arquivo de configuração do grub
```
update-grub
```

### Saindo do chroot e desmontando tudo.
```
exit

umount sys
umount proc
umount dev
cd
umount /novo
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

(hd0) (hd0,msdos1) (hd1) (hd1,msdos2) (hd1,msdos1)
```
Isso mostrará os discos que tem. Se rodar `ls (hd0,msdos1)/` verá o conteúdo da partição que está fora do raid.\
Se rodar `ls (hd1,msdos2)/` deve obter uma mensagem de erro, pois o grub ainda não é capaz de entender o raid.
Vamos carregar o módulo do grub que dá essa capacidade:
```
ismod mdraid1x
```
Agora rodando `ls` verá mais um dispositivo, que é o raid.
```
ls
(proc) (hd0) (hd0,msdos1) (hd1) (hd1,msdos2) (hd1,msdos1) (md/1)
```
Vamos carregar o arquivo de configuração do grub que está no disco em raid e dar boot por ele.
```
configfile (md/1)/boot/grub/grub.cfg
```
Você verá a tela do grub, já com as configurações do raid, caso queira conferir tecle e.

## Etapa 7 (Incluindo disco1 ao raid)

Logue no sistema para continuar.

### Checagem de pontos de montagem
```
lsblk

NAME    MAJ:MIN RM SIZE RO TYPE  MOUNTPOINT
vda     254:0    0   7G  0 disk
└─vda1  254:1    0   7G  0 part
vdb     254:16   0   7G  0 disk
├─vdb1  254:17   0   1M  0 part
└─vdb2  254:18   0   7G  0 part
  └─md1   9:1    0   7G  0 raid1 /
```
Repare que o / está montado no raid1 e que a partição vda1 está desmontada.

### Copiar a tabela de partições para o primeiro disco
Vamos usar o programa *sgdisk* do pacote gdisk, caso não o tenha instalado rode:
```
apt install gdisk
```

Copiando a tabela de partições.
Criando um backup
```
sfdisk -d /dev/vdb > bkp
```
Usando o backup para replicar no primeiro disco.

```
sfdisk /dev/vda < bkp

Verificando se ninguém está usando este disco no momento ... OK

Disco /dev/vda: 7 GiB, 7516192768 bytes, 14680064 setores
Unidades: setor de 1 * 512 = 512 bytes
Tamanho de setor (lógico/físico): 512 bytes / 512 bytes
Tamanho E/S (mínimo/ótimo): 512 bytes / 512 bytes
Tipo de rótulo do disco: dos
Identificador do disco: 0x29e236dd

Situação antiga:

Dispositivo Inicializar Início      Fim  Setores Tamanho Id Tipo
/dev/vda1   *             2048 14680063 14678016      7G 83 Linux

>>> Cabeçalho de script aceito.
>>> Cabeçalho de script aceito.
>>> Cabeçalho de script aceito.
>>> Cabeçalho de script aceito.
>>> Criado um novo rótulo de disco DOS com o identificador de disco 0xdc578e40.
/dev/vda1: Criada uma nova partição 1 do tipo "Non-FS data" e de tamanho 1 MiB.
Partição nº 1: contém uma assinatura de ext4.
/dev/vda2: Criada uma nova partição 2 do tipo "Linux" e de tamanho 7 GiB.
/dev/vda3: Concluído.

Situação nova:
Tipo de rótulo do disco: dos
Identificador do disco: 0xdc578e40

Dispositivo Inicializar Início      Fim  Setores Tamanho Id Tipo
/dev/vda1                 2048     4095     2048      1M da Dados Não-FS
/dev/vda2                 4096 14680063 14675968      7G 83 Linux

A tabela de partição foi alterada.
Chamando ioctl() para reler tabela de partição.
Sincronizando discos.
```

### incluir a partição do primeiro disco ao raid1
```
mdadm /dev/md1 -a /dev/vda2
```

### Verificando o progresso da sincronização do array
```
cat /proc/mdstat
Personalities : [raid1] [linear] [multipath] [raid0] [raid6] [raid5] [raid4] [raid10]
md1 : active raid1 vda2[2] vdb2[1]
      7332864 blocks super 1.2 [2/1] [_U]
      [==>..................]  recovery = 13.6% (1000832/7332864) finish=0.5min speed=200166K/sec

unused devices: <none>

```
Rode o comando mais de uma vez até que termine a sincronização, verá assim:
```
cat /proc/mdstat

Personalities : [raid1] [linear] [multipath] [raid0] [raid6] [raid5] [raid4] [raid10]
md1 : active raid1 vda2[2] vdb2[1]
      7332864 blocks super 1.2 [2/2] [UU]

unused devices: <none>

```
Raid1 ok, veja que agora mostra `[UU]` evidenciando a integridade do raid1.

### Veja os dispositivos agora
```
lsblk

NAME    MAJ:MIN RM SIZE RO TYPE  MOUNTPOINT
vda     254:0    0   7G  0 disk
├─vda1  254:1    0   1M  0 part
└─vda2  254:2    0   7G  0 part
  └─md1   9:1    0   7G  0 raid1 /
vdb     254:16   0   7G  0 disk
├─vdb1  254:17   0   1M  0 part
└─vdb2  254:18   0   7G  0 part
  └─md1   9:1    0   7G  0 raid1 /
```

## Etapa 8 (Preparando boot)

### Ajustando o grub
```
update-grub
```

### **O bizu!**
Vamos gravar cópias do grub em cada um dos discos, nesse caso não dará erros porque a partição de 1M que criamos é justamente para acomodar o grub.
```
grub-install /dev/vda
grub-install /dev/vdb
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

