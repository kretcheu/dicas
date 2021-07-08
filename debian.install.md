### Preparar uma partição com instalador do Debian

A ideia surgiu no grupo de Debian por JamersonG(7)

1. crie uma partição de tamanho suficiente para a imagem de instalação que escolheu.
   - 400Mb se for Netinst
   - 4G se for DVD-1.

Digamos que seja a partição 3 a título de exemplo:

2. formatar ext4 essa partição.

```
mkfs.ext4 /dev/sda3
```

3. Montar essa partição em /mnt
```
mount /dev/sda3 /mnt
```

4. Montar a imagem ISO do Debian em /media
```
mount /caminho/arquivo/debian.iso /media
```

5. Sincronizar o conteúdo:
```
rsync -av /media /mnt
```

6. desmontar /mnt e /media
```
umount /mnt
umount /media
```

7. No seu grub colocar uma entrada para o instalador

Para isso edite /etc/grub.d/40_custom
com o conteúdo:

```
#!/bin/sh
exec tail -n +3 $0
# This file provides an easy way to add custom menu entries. Simply type the
# menu entries you want to add after this comment. Be careful not to change
# the 'exec tail' line above.

menuentry "Debian Install" {
insmod part_gpt
insmod part_msdos
set root=(hd0,gpt3)
configfile /boot/grub/grub.cfg
}

Dê permissão de execução:
```
chmod +x /etc/grub.d/40_custom
```
OBS.:

Ajuste a linha *set root* para o disco e partição correspondente a nova partição.
exemplos:

- GPT
set root=(hd0,gpt3)
set root=(hd1,gpt4)

- MBR
set root=(hd0,msdos3)

Criar o novo arquivo de configuração do GRUB.
```
update-grub
```

Agora quando der boot terá a entrada "Debian Install" para o instalador.


