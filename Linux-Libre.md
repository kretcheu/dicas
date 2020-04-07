
# Instalando o Kernel Linux-Libre no Debian

DISCLAIMER MEGA IMPORTANTE!!!

> Esta página será atualizada sempre que houver uma mudança na documentação oficial localizada em: **[https://jxself.org/linux-libre/](https://web.archive.org/web/20180913024046/https://jxself.org/linux-libre/)**

O projeto linux-libre nasceu da necessidade da comunidade de manter em seus sistemas GNU um kernel livres de blobs proprietários, o linux-libre é o resultado do trabalho dessa comunidade, entregando assim um kernel 100% livre. Hoje nós vamos ver como instalar ele no Debian Jessie (e derivados).

[![](https://web.archive.org/web/20180913024046im_/https://www.fsfla.org/ikiwiki/selibre/linux-libre/100gnu+freedo.png)](https://web.archive.org/web/20180913024046/https://www.fsfla.org/ikiwiki/selibre/linux-libre/)

As always, clique na imagem para conhecer a página do projeto.

As instruções que irei passar aqui e demonstrarei em vídeo vem deste link aqui: **[https://jxself.org/linux-libre/](https://web.archive.org/web/20180913024046/https://jxself.org/linux-libre/)** e caso alguém tenha alguma dúvida, basta dar um pulo lá.

Para usar o repositório citado neste tutorial é preciso instalar o pacote: `apt-transport-https`:

    apt install apt-transport-https

Abrir a lista de sources com:

    apt edit-sources

adicionar a linha:

    deb https://linux-libre.fsfla.org/pub/linux-libre/freesh/ freesh main

Você deve também instalar a chave GPG deste repositório:

    wget https://jxself.org/gpg.inc

Verifique se é a chave correta:

    gpg --keyid-format long --with-fingerprint gpg.inc

Tenha **CERTEZA** que a saída é essa:

Key fingerprint = F611 A908 FFA1 65C6 9958 4ED4 9D0D B31B 545A 3198

Se bater a saída, configure o gerenciador de pacotes para confiar na chave e apague a cópia local em seguida:

    apt-key add < gpg.inc
    rm gpg.inc

Agora você poderá atualizar o gerenciador de pacotes e instalar o Linux-Libre:

    apt update

Para sistemas X86 (32bits):

    apt install linux-libre32

Para sistemas X64 (64bits):.

    apt install linux-libre

E se você estiver usando libreboot, não se esqueça de:

    cd /boot/grub
    ln -s grub.cfg libreboot_grub.cfg

E é isso turma. Como falei, esta página será sempre atualizada conforme o processo for mudando, e é meramente uma tradução do conteúdo original.

Ah, e eu não costumo usar sudo no Debian, eu entro como usuário Root e realizo as ações... então eu retirei as menções ao sudo na lista de comandos. =)

Fiquem na paz e até mais!

O trabalho **Instalando o Kernel Linux-Libre no Debian** 
de [Thiago Faria Mendonça](https://web.archive.org/web/20180913024046/http://acesso.me/acesso/)  
está licenciado com uma Licença
[Creative Commons – Atribuição 4.0 Internacional](https://web.archive.org/web/20180913024046/https://creativecommons.org/licenses/by/4.0/).


