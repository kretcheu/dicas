## Cola Faxina no Debian

### Atualização

    apt update
    apt upgrade
    apt full-upgrade

### Autoremove

    apt autoremove

### Pacotes órfãos

    apt install deborphan

    deborphan
    deborphan --guess-all
    deborphan -a

### Integridade dos pacotes

    apt install debsums

    debsums -a 1>log 2>erros

    cat erros
    apt reinstall pacote

    grep -v OK log

    - arquivos alterados
    - arquivos faltando

    exemplo: foremost

    apt reinstall -o Dpkg::Options::=--force-confmiss pacote

### Pacotes de fora do Debian

    aptitude search '~S~i!~Odebian'
    apt list '~i!~Odebian'

    vrms
    apt list '~i~snon-free'
    apt list --installed non-free

### Kernel antigos

    dpkg -l | grep linux-image
    dpkg -l | grep linux-headers

    apt purge linux-image-5.9.0-3-amd64

    ls /lib/modules

### Xserver-xorg outras placas

    dpkg -l | grep xserver-xorg-video

    apt purge xserver-xorg-video-nouveau

### Cache de pacotes

    ls -l /var/cache/apt/archives/
    apt clean

    rm /var/cache/apt/archives/*deb
    apt install pacote -d

### Logs antigos

    cd /var/log/

    find | grep gz$|xargs rm
    find | grep 1$|xargs rm
    find | grep old$|xargs rm

### Backups antigos

    cd /var/backups

    arquivos gz

### Dados do usuário

- Cache

   ~/.cache

- Arquivos de configuração

   ~/.
   ~/.config


