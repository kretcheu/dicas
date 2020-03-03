# Debootstrap com grml-debootstrap


## Preparando...

### Instalando o grml-debootstrap
```
apt install grml-debootstrap
```

### Criando as partições
```
fdisk /dev/vdc

Resultado:

Disk /dev/vdc: 6 GiB, 6442450944 bytes, 12582912 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 0CCFB3F6-00E2-FA43-A391-2306F80D91D7

Device      Start      End  Sectors Size Type
/dev/vdc1    2048   104447   102400  50M EFI System
/dev/vdc2  104448 12582878 12478431   6G Linux filesystem
```

## Fazendo instalação
```
grml-debootstrap --efi /dev/vdc1 --target /dev/vdc2 --grub /dev/vdc --hostname deboot


 * EFI support enabled now.
 * grml-debootstrap [0.89] - Please recheck configuration before execution:

   Target:          /dev/vdc2
   Install grub:    /dev/vdc
   Install efi:     /dev/vdc1
   Using release:   buster
   Using hostname:  deboot
   Using mirror:    http://deb.debian.org/debian
   Using arch:      amd64
   Config files:    /etc/debootstrap

   Important! Continuing will delete all data from /dev/vdc2!

 * Is this ok for you? [y/N]

```
## Ao digitar 'y' começa o processo...
```
Logical sector size is zero.
 * EFI partition /dev/vdc1 doesn't seem to be formatted, creating filesystem.
mkfs.fat 4.1 (2017-01-24)
mkfs.fat: warning - lowercase labels might not work properly with DOS or Windows
 * Running mkfs.ext4  on /dev/vdc2
mke2fs 1.44.5 (15-Dec-2018)
Discarding device blocks: done                            
Creating filesystem with 1559803 4k blocks and 390144 inodes
Filesystem UUID: 890488fd-35af-4be3-97a1-caff8ae54818
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 

 * Disabling automatic filesystem check on /dev/vdc2 via tune2fs
tune2fs 1.44.5 (15-Dec-2018)
Setting maximal mount count to -1
Setting interval between checks to 0 seconds
 * Mounting /dev/vdc2 to /mnt/debootstrap.1344
 * Identified UUID 890488fd-35af-4be3-97a1-caff8ae54818 for /dev/vdc2
 * Running debootstrap  for release buster (amd64) using http://deb.debian.org/debian
 * Executing: debootstrap --arch amd64   buster /mnt/debootstrap.1344 http://deb.debian.org/debian
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

vários outros pacotes...


Adding boot menu entry for EFI firmware configuration
done
   Executing stage passwords
Activating shadow passwords.
Shadow passwords are now on.
Setting password for user root:
Enter new UNIX password for user root: 
Retype new UNIX password for user root: 
   Executing stage custom_scripts
   Executing stage upgrade_system
Running update + upgrade

Atualizando ....

Pronto!


```



