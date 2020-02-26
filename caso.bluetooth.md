# Bluetooth não funcionou

### Vamos descobrir qual o dispositivo

    lsusb

    Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
    Bus 001 Device 005: ID 0cf3:e005 Qualcomm Atheros Communications
    Bus 001 Device 004: ID 0bda:0129 Realtek Semiconductor Corp. RTS5129 Card Reader Controller
    Bus 001 Device 003: ID 0c45:6712 Microdia
    Bus 001 Device 002: ID 248a:8367 Maxxter
    Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

Tudo indica que é esse: **ID 0cf3:e005 Qualcomm Atheros Communications**

Parece ser o caso de falta de firmware, vamos averiguar:

    grep firmware /var/log/syslog | grep failed

    Feb 24 10:24:00 Debian kernel: [   13.474673] usb 1-8: firmware: failed to load ar3k/AthrBT_0x31010000.dfu (-2)

Isso quer dizer que o arquivo que falta é: **AthrBT_0x31010000.dfu**

Pesquisando no site do Debian:

- [https://www.debian.org/distrib/packages](https://www.debian.org/distrib/packages)


        Use o formulário: **"Procurar o conteúdo dos pacotes"**
        colocando o nome do arquivo no campo: **"Palavra-Chave"** e
        selecione sua versão de Debian.

O resultado é que esse arquivo está no pacote: **firmware-atheros**

Para instalar o pacote do firmware há dois modos:

## Métodos de instalação


* Método (**X**) Incluir as seções contrib e non-free.

   Usando seu editor de texto preferido edite o arquivo: **/etc/apt/sources.list** e rode:

    apt update
    apt install firmware-atheros


Exemplo de sources.list:

    deb http://deb.debian.org/debian buster main contrib non-free
    deb http://deb.debian.org/debian buster-updates main contrib non-free
    deb http://security.debian.org buster/updates main contrib non-free


* Método (**Y**) Baixar o pacote separadamente, usando outro sistema ou caso tenha conectividade via cabo usando o mesmo.

- [Pacote firmware-atheros](http://ftp.us.debian.org/debian/pool/non-free/f/firmware-nonfree/firmware-atheros_20190114-2_all.deb)

* Depois do arquivo baixado rode:


        apt install firmware-atheros_20190114-2_all.deb
        ou
        dpkg -i firmware-atheros_20190114-2_all.deb



- A mais feia rebootando o sistema. (**Rebootar "Jamais"!**)
- A mais elegante rodando:


        modprobe -r ath3k
        modprobe ath3k


