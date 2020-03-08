# Swap Criptografado

## Numa instalação de Debian

## Vendo o particionamento
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
```

## Desativando swap
`swapoff -a`

## Colocando uma entrada no /etc/crypttab.
```
# <target name> <source device> <key file>      <options>
cswap          /dev/vda2       /dev/urandom    swap,cipher=aes-xts-plain64,size=256,hash=sha1
```

## Mudando o swap no /etc/fstab
É necessário tracar a entrada do swap no /etc/fstab
de:
```
/dev/vda2          none        swap    sw            0       0
```
por:
```
/dev/mapper/cswap  none        swap    sw            0       0
```

## Instalar cryptsetup-run
```
apt install cryptsetup-run
```

## Para reativar o swap
```
cryptdisks_start cswap
swapon -a
```

## Verificar se habillitou
```
swapon --summary
Filename				Type		Size	Used	Priority
/dev/dm-0                              	partition	5242876	0	-2

free
              total        used        free      shared  buff/cache   available
Mem:         988072      134404      637316       12788      216352      700720
Swap:       5242876           0     5242876

```


## Desabilitar o RESUME
```
echo "RESUME=none" >/etc/initramfs-tools/conf.d/resume

update-initramfs -u -k all
```

## Boot não prossegue
Em alguns casos, não sei ainda quais, depois de habilitar o swap criptografado o boot fica parado.   
Para esses casos o workaround a seguir resolve.

### Criando um gatilho para não dar problema de parar no boot
Repare que o nome do serviço varia de acordo com o nome que usou para criar o swap.
```
mkdir /etc/systemd/system/systemd-cryptsetup@cswap.service.d/
```

## Criando o arquivo 90-trigger-udev.conf.
Nesse diretório crie um arquivo de configuração, por exemplo chamado: 90-trigger-udev.conf, nele o seguinte conteúdo.
```
# Run udevadm trigger after the mkswap call in the original generated
# service

[Service]
ExecStartPost=/sbin/udevadm trigger /dev/mapper/%i
```
