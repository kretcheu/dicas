## Boot sem GRUB
Nesse tutorial vamos mostrar como usar a UEFI para dar boot sem precisar um bootloader do sistema operacional como o GRUB.

Os PCs fabricados a partir de 200X costumam já possuir UEFI, que veio em substituição a antiga BIOS.\
A UEFI possui vários recursos que não existiam na BIOS.

 - Conhece o sistema de particionamento GPT e DOS.
 - Conhece o sistema de arquivos FAT.
 - É capaz de executar programas compilados no formato PE.
 - É capaz de carregar o kernel e o initrd, dispensando a necessidade do bootloader.
 - Algumas implementam o *Secureboot*, pois podem verificar assinaturas digitais do que vai ser carregado.

Para que a UEFI possa carregar o *kernel* e o *initramfs* é necessário que esses arquivos estejam na partição EFI.

Essa partição costuma ser a primeira partição do disco e é a mesma onde são colocados os bootloaders dos sistemas operacionais.

Essa partição tem o sistema de arquivos FAT32 ou como diriam alguns "foi formatada como FAT32".

### Descrição das etapas

Vamos fazer os seguintes passos:

1. Copiar kernel e initramfs.
2. Criar uma entrada na UEFI.
3. Preparar para novas versões de kernel.

### Etapa 1 (Copiar kernel e initramfs)
Vamos copiar os arquivos do kernel e do initramfs para o diretório correspondente a partição EFI.\
Por padrão essa partição é montada em **/boot/efi**.
```
cp /boot/vmlinuz-4.19.0-8-amd64 /boot/efi/EFI/vmlinuz
cp /boot/initrd.img-4.19.0-8-amd64 /boot/efi/EFI/initrd.img
```
Adapte essaas linhas para a versâo de kernel correspondente a sua instalação, usar a tecla **TAB** ajuda bem!\
Veja que no nome dos arquivos destinos não usei o número da versão (4.19.0) no meu caso.\
Desse modo fica mais fácil criar a entrada UEFI e para que quando formos preparar para as novas versões de kernel não seja preciso refazer a entrada UEFI.

### Etapa 2 (Criar uma entrada na UEFI)
Na entrada UEFI é preciso passar também os parâmetros para o kernel, assim como é feito no bootloader.

Como saber quais parâmetros são necessários no seu caso?
Vamos verificar o que está no arquivo de configuração do grub.
```
grep vmlinuz  /boot/grub/grub.cfg
	linux	/boot/vmlinuz-5.6.3-gnu root=UUID=336dbd2e-3e4f-4fba-9a01-6ea17e8b1802 ro  quiet
	linux	/boot/vmlinuz-4.19.98 root=UUID=336dbd2e-3e4f-4fba-9a01-6ea17e8b1802 ro  quiet
	linux	/boot/vmlinuz-4.19.0-8-amd64 root=UUID=336dbd2e-3e4f-4fba-9a01-6ea17e8b1802 ro  quiet
```

Como nessa máquina tenho mais de uma versão de kernel instalada, várias linhas são mostradas no resultado do comando.\
A que nos interessa é a da versão *4.19.0-8* a mesma que copiamos os arquivos.

Quem coloca a entrada na UEFI é o programa efibootmgr.\
Se você está usando UEFI certamente o pacote já está instalado.

```
efibootmgr -c -d /dev/sda -p 1 -L "Debian (sem GRUB)" -l /EFI/vmlinuz -u "root=UUID=336dbd2e-3e4f-4fba-9a01-6ea17e8b1802 ro quiet initrd=/EFI/initrd.img"
```
Entendendo cada parte da linha:
```
-c    - cria uma nova variável e adiciona as entradas UEFI.
-d    - indica o dispositivo.
-p 1  - indica qual a partição UEFI.
-L    - define o nome da entrada.
-l    - define o que vai ser carregado.
-u    - define os parâmetros a serem passados.
```

Para verificar as entradas UEFI rode:
```
efibootmgr -v
```
Pronto a entrada está lá e você já pode testar.

Para remover entradas, alterar a ordem de boot deve usar o efibootmgr:

Para remover a entrada 0004:
```
efibootmgr -b 4 -B
```

Para definir a ordem:
```
efibootmgr -o 0004,0002,0001,0000,0003
```

Para saber mais sobre efibootmgr:
```
efibootmgr --help
man efibootmgr
```
Se quiser que as novas versões de kernel já estejam disponíveis de modo automatizado, siga a próxima etapa.

### Etapa 3 (Preparar para novas versões de kernel)
Para automatizar o processo e a cada nova versão de kernel os arquivos sejam atualizados, crie um arquivo chamado:\
`/etc/kernel/postinst.d/zz-update-efi` com o conteúdo:

```
#!/bin/sh -e

version="$1"

echo "Updating EFI boot files..."

cp /boot/vmlinuz-${version} /boot/efi/EFI/vmlinuz
cp /boot/initrd.img-${version} /boot/efi/EFI/initrd.img
```

Dê permissão de execução:
```
chmod +x /etc/kernel/postinst.d/zz-update-efi
```

### Conclusão
Essa metodologia serve para poder carregar o kernel e o initramfs sem precisar do bootloader do sistema.\
Espero que tenha conseguido seguir as etapas e conseguido dar boot no seu Debian.

Qualquer dúvida me procure no Telegram!

- [Grupo Curso GNU](https://t.me/cursognu)


