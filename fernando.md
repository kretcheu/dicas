## Resolvendo a bronca da Nvidia

* Criando o ponto de montagem

        mkdir /debian

* Montando a partição onde está o sistema

        mount /dev/sda1 /debian

* Montando os diretórios virtuais para poder chrootar

        mount --bind /proc /debian/proc
        mount --bind /dev /debian/dev
        mount --bind /dev/pts /debian/dev/pts
        mount --bind /sys /debian/sys

* chrootando

        chroot /debian

* Atualizando lista de pacotes

        apt update

* Atualizando

        apt upgrade

* Removendo pacotes desnecessários

        apt autoremove

* Resolvendo problema de locales

        apt install locales
        dpkg-reconfigure locales

**Selecione apenas pt-br-utf8 e faça que seja o defaut**

* Como deu erro nas placas é legal a gente verificar se tem algum arquivo faltando, para isso vamos instalar uma ferramenta:

        apt install debsums

* Testando tudo que está instalado.

        debsums -a 1>log 2>err

>Esse demora um pouquinho tb
>Não vai aparecer nenhum resultado na tela ainda.

* Vendo se tem erros

        cat err

* Vendo se e quais arquivos estão alterados

        grep -v OK log

* Reinstalando pacotes alterados.
Não foi necessário.

* Verificando as versões de kernel

        dpkg -l | grep linux-image


        ii  linux-image-4.19.0-6-amd64                4.19.67-2+deb10u2                   amd64        Linux 4.19 for 64-bit PCs (signed)
        ii  linux-image-4.19.0-8-amd64                4.19.98-1                           amd64        Linux 4.19 for 64-bit PCs (signed)
        ii  linux-image-amd64                         4.19+105+deb10u3                    amd64        Linux for 64-bit PCs (meta-package)


*  Verificando UEFI

         efibootmgr -v


         bash: efibootmgr: command not found

     Não está UEFI mas não há problema nisso.


* Verificando a interface de rede Wi-Fi.

        lsusb


        Bus 002 Device 003: ID 0bda:8178 Realtek Semiconductor Corp. RTL8192CU 802.11n WLAN Adapter
        Bus 002 Device 002: ID 0c45:627b Microdia PC Camera (SN9C201 + OV7660)
        Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
        Bus 004 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
        Bus 001 Device 007: ID 0781:5567 SanDisk Corp. Cruzer Blade
        Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
        Bus 003 Device 003: ID 058f:6362 Alcor Micro Corp. Flash Card Reader/Writer
        Bus 003 Device 002: ID 1a2c:2c27 China Resource Semico Co., Ltd
        Bus 003 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub


        lspci -nnkd::0200


        06:00.0 Ethernet controller [0200]: Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller [10ec:8168] (rev 01)
        Subsystem: Elitegroup Computer Systems RTL8111/8168 PCI Express Gigabit Ethernet controller [1019:8168]
        Kernel driver in use: r8169
        Kernel modules: r8169


* Ajustando sources.list

        deb http://deb.debian.org/debian buster main contrib non-free
        deb http://deb.debian.org/debian buster-updates main contrib non-free
        deb http://security.debian.org buster/updates main contrib non-free

        apt update
        apt upgrade

*  Vamos refazer o initrd

        update-initramfs -u -k all


Faltou firmware, workaround:

      mkdir /lib/firmware/nvidia/gv100
      cd /lib/firmware/nvidia/gp100
      cp -r  acr/ gr/ ../gv100/
      cd ../gp107
      cp -r  sec2/ nvdec/ ../gv100/


Rodando novamente:

      update-initramfs -u -k all


* Desmontando tudo


        exit
        umount /debian/proc
        umount /debian/dev/pts
        umount /debian/dev
        umount /debian/sys
        umount /debian

        shutdown -r now


* Testar o sistema!!

