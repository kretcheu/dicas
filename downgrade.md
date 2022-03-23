# Tutorial de downgrade de um pacote

Tutorial de downgrade de um pacote para resolver um Bug depois de uma atualização.

Em vídeo: https://www.youtube.com/watch?v=x2JQaXfI-pI

## O problema

Ao tentar reproduzir um vídeo deu erro de falha de segmentação (segfault)

```
mpv aula36.mp4
 (+) Video --vid=1 (*) (h264 1280x720 29.970fps)
 (+) Audio --aid=1 (*) (aac 2ch 48000Hz)
Falha de segmentação

vlc aula36.mp4
VLC media player 3.0.17.3 Vetinari (revision 3.0.13-8-g41878ff4f2)
[0000556c3f9975b0] main libvlc: Executando o VLC com a interface padrão. Use 'cvlc' para usar o VLC sem interface.
[00007f17ac0043a0] gl gl: Initialized libplacebo v4.192.1 (API v192)
libva info: VA-API version 1.14.0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/iHD_drv_video.so
libva info: Found init function __vaDriverInit_1_14
Falha de segmentação
```

## Descobrindo o responsável

Instalando gdb

```
apt install gdb
```

Rodando o programa com gdb

```
gdb
r aula36.mp4

Thread 14 "mpv/vo" received signal SIGSEGV, Segmentation fault.
[Switching to Thread 0x7fffc7296640 (LWP 1107955)]
0x00007fffc73c69ae in ?? () from /usr/lib/x86_64-linux-gnu/libigdgmm.so.12
ctrl-d
y
```

## Descobrindo o pacote

```
dpkg -S /usr/lib/x86_64-linux-gnu/libigdgmm.so.12
libigdgmm12:amd64: /usr/lib/x86_64-linux-gnu/libigdgmm.so.12
```

## Incluindo o repositório bookworm

Incluindo a linha no /etc/apt/sources.list

deb https://deb.debian.org/debian bookworm main

## Atualizando a lista de pacotes

```
apt update
```

## Verificando as versões disponíveis do pacote

```
apt policy libigdgmm12
libigdgmm12:
  Instalado: 22.1.1+ds1-1
  Candidato: 22.1.1+ds1-1
  Tabela de versão:
 *** 22.1.1+ds1-1 500
        500 https://deb.debian.org/debian sid/main amd64 Packages
        100 /var/lib/dpkg/status
     22.0.2+ds1-1 500
        500 https://deb.debian.org/debian bookworm/main amd64 Packages
```

## Fazendo o downgrade

```
apt install libigdgmm12/bookworm
```

Verificando com policy

```
apt policy libigdgmm12
libigdgmm12:
  Instalado: 22.0.2+ds1-1
  Candidato: 22.1.1+ds1-1
  Tabela de versão:
     22.1.1+ds1-1 500
        500 https://deb.debian.org/debian sid/main amd64 Packages
 *** 22.0.2+ds1-1 500
        500 https://deb.debian.org/debian bookworm/main amd64 Packages
        100 /var/lib/dpkg/status
```
## Marcando o pacote como mantido

```
apt-mark hold libigdgmm12
```

## Se inscrevendo no bug

Envie um email para: 1007992-subscribe@bugs.debian.org

## Depois de solucionado o problema no pacote

```
apt-mark unhold libigdgmm12
apt update
apt upgrade
```
