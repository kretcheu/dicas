# Transformar um live-cd numa instalação regular.

1. Preparativos
  - Baixar imagem ISO.
  - Escolher partições

2. Montar
  - ISO
  - squash
  - partições

3. sincronizar

4. Ajustes
  - chroot
  - criar usuário
  - instalar grub
  - atualizar sistema

5. Ajustes finais

### Etapa 1 (Preparativos)

#### Baixar imagem ISO.

Acesse o site de download de imagens do Debian, para escolher qual imagem usar.\
Há imagens de live-cd para arquitetura 64bits (amd64) com diversos ambientes de desktop diferentes.

Imagens para 64 bits: [amd64][debian-amd64]

Vou usar como exemplo o live-cd com Mate, mas escolha o da sua preferência.\
Rodando seu Debian, abra um terminal para baixar o arquivo iso e os hashs.
```
wget https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/debian-live-10.3.0-amd64-mate.iso
wget https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/SHA1SUMS
```

Completado download vamos verificar a integridade do arquivo de imagem iso.

```
sha1sum -c SHA1SUMS --ignore-missing
```

#### Escolher partições

Nesse momento temos que escolher o destino da instalação.

- Uma partição do seu HD.
- Um outro HD ou pendrive.
- Um disco virtual.

Qualquer que seja sua escolha, a ideia será a mesma, aliás como em qualquer instalação.

- Copiar arquivos.
- Preparar para bootar.

Nesse exemplo vou usar um HD externo. Se estiver querendo fazer de outro modo e tenha dúvidas pode me procurar, veja no final os meios de contato.

O HD que escolhi tem as seguintes características:
```
Disk /dev/vdc: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: A92B0DD4-DE14-B041-A7D7-5835F8251CF4

Device      Start      End  Sectors Size Type
/dev/vdc1    2048   104447   102400  50M EFI System
/dev/vdc2  104448 20971486 20867039  10G Linux filesystem
```

### Etapa 2 (montar)

#### Montar ISO
```
mkdir /tmp/iso
mkdir /tmp/live-cd
```

#### Montar squash
```
mount debian-live-10.3.0-amd64-mate.iso /tmp/iso
mount /tmp/iso/live/filesystem.squashfs /tmp/live-cd
```

#### Montar partições
```
mkdir /tmp/debian
mount /dev/vdc2 /tmp/debian
mkdir -p /tmp/debian/boot/efi
mount /dev/vdc1 /tmp/debian/boot/efi
```

### Etapa 3 (sincronizar)
```
rsync -av /tmp/live-cd /tmp/debian
```

### Etapa 4 (Ajustes)

#### Chroot
```
cd /tmp/debian

mount --bind /proc proc
mount --bind /dev dev
mount --bind /sys sys

chroot .
```

#### Criar usuário
```
adduser user
adduser user sudo

```
#### Instalar GRUB
```
apt install grub-efi
```
### Instalar o GRUB no dispositivo

```
grub-install /dev/vdc
```

## Atualizar arquivo de configuração do grub
```
update-grub
```

#### Atualizar
```
apt update
apt upgrade
```

#### Sair do chroot
```
exit ou ctrl-d
```

#### Ajuste do fstab
Agora vamos ajustar o */etc/fstab*.\
Quem faz muito bem esse papel é um programa chamado `genfstab` que está no pacote **arch-install-scripts**.\

Caso ainda não tenha instalado, vamos instalar.
```
apt install arch-install-scripts

genfstab -U /tmp/debian >/tmp/debian/etc/fstab

```

#### Desmontar partições

```
cd

umount /tmp/debian/boot/efi
umount /tmp/debian/proc
umount /tmp/debian/dev
umount /tmp/debian/sys
umount /tmp/debian
```

### Conclusão
Essa metodologia serve para transferir qualquer distribuição live-cd, eventualmente com alguns pequenos ajustes.\
Já fiz com Kali, PureOS e outros.

Qualquer dúvida me procure no Telegram!

- [Grupo Debian Brasil](https://t.me/debianbrasil)
- [Grupo Debian BR](https://t.me/debianbr)


