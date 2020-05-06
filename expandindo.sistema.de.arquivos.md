# Expandindo Sistema de Arquivos ext4

A ideia é a seguinte:
Tenho um HD com algumas partições, uma deles sem uso, como fazer para encorporar essa partição sem uso a outra e aprovitar do espaço.

# O estado atual
```
fdisk -l /dev/vda
Disk /dev/vda: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2EA7B5C3-1854-0C47-950B-C48585611FC1

Device        Start      End  Sectors Size Type
/dev/vda1      2048   104447   102400  50M EFI System
/dev/vda2    104448 31457246 31352799  15G Linux filesystem
/dev/vda3  31457280 41943006 10485727   5G Linux swap


df -h
Sist. Arq.      Tam. Usado Disp. Uso% Montado em
udev            464M     0  464M   0% /dev
tmpfs            97M  3,5M   94M   4% /run
/dev/vda2        15G  7,6G  6,3G  55% /
tmpfs           483M     0  483M   0% /dev/shm
tmpfs           5,0M  4,0K  5,0M   1% /run/lock
tmpfs           483M     0  483M   0% /sys/fs/cgroup
/dev/vda1        50M   13M   38M  26% /boot/efi
tmpfs            97M  4,0K   97M   1% /run/user/107
tmpfs            97M     0   97M   0% /run/user/0

```

Bem, como resolvi usar swap num arquivo, pretendo apagar a partição de swap (que não estou usando) e
aproveitar o espaço na minha partição do /.

Com transplante isso é moleza, mas dessa vez quis tentar "on the fly" ou seja com o sistema rodando, sem usar live-cd e essas coisas, vamos ver no que dá.\
Não custa recomendar em negrito e maiúsculas: **FAÇA UM BACKUP DO DISCO TODO**.

Como se tratava de uma máquina virtual eu desliguei a VM e fiz o bakup do arquivo que representa o disco virtual.

Agora vou apagar a partição 3 que um dia foi o swap, mas que agora está sem uso.

Usei o fdisk, assim:
```
fdisk /dev/vda

Welcome to fdisk (util-linux 2.33.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): p
Disk /dev/vda: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2EA7B5C3-1854-0C47-950B-C48585611FC1

Device        Start      End  Sectors Size Type
/dev/vda1      2048   104447   102400  50M EFI System
/dev/vda2    104448 31457246 31352799  15G Linux filesystem
/dev/vda3  31457280 41943006 10485727   5G Linux swap

Command (m for help): d
Partition number (1-3, default 3):3

Partition 3 has been deleted.

Command (m for help): w
The partition table has been altered.
Syncing disks.
```

Para evitar dores de cabeça rodei partprobe para o kernel reler a tabela de partições.
```
partprobe

fdisk /dev/vda -l
Disk /dev/vda: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2EA7B5C3-1854-0C47-950B-C48585611FC1

Device      Start      End  Sectors Size Type
/dev/vda1    2048   104447   102400  50M EFI System
/dev/vda2  104448 31457246 31352799  15G Linux filesystem
```

Parte da "mágica" é feita na tabela de partições, quem vai cuidar disso é um programa chamdo growpart que está no **cloud-guest-utils**,
Outra parte é feita no sistema de arquivos com um programa chamado resizefs.

```
apt install cloud-guest-utils
```

Como a partição que quero que cresca é a 2 o comando fica:
```
growpart /dev/vda 2

CHANGED: partition=2 start=104448 old: size=31352799 end=31457247 new: size=41838559,end=41943007

fdisk /dev/vda -l

Disk /dev/vda: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2EA7B5C3-1854-0C47-950B-C48585611FC1

Device      Start      End  Sectors Size Type
/dev/vda1    2048   104447   102400  50M EFI System
/dev/vda2  104448 41943006 41838559  20G Linux filesystem
```

Partição aumentada, resta ajustar o sistema de arquivos para aproveitar esse novo espaço.

```
resize2fs /dev/vda2

resize2fs 1.44.5 (15-Dec-2018)
Filesystem at /dev/vda2 is mounted on /; on-line resizing required
old_desc_blocks = 2, new_desc_blocks = 3
The filesystem on /dev/vda2 is now 5229819 (4k) blocks long.
```

Rodando `df -h`novamente para comparar com o inicial.
```
df -h

Sist. Arq.      Tam. Usado Disp. Uso% Montado em
udev            464M     0  464M   0% /dev
tmpfs            97M  3,5M   94M   4% /run
/dev/vda2        20G  7,6G   12G  41% /
tmpfs           483M     0  483M   0% /dev/shm
tmpfs           5,0M  4,0K  5,0M   1% /run/lock
tmpfs           483M     0  483M   0% /sys/fs/cgroup
/dev/vda1        50M   13M   38M  26% /boot/efi
tmpfs            97M  4,0K   97M   1% /run/user/107
tmpfs            97M     0   97M   0% /run/user/0
```

Removendo a referência ao swap

Remova o arquivo `/etc/initramfs-tools/conf.d/resume`
Para ajustar o initrd a essa nova situação rode:
```
update-initramfs -u -k all
```


