# Usando swap em arquivo criptografado

### Surpresas!?
Para pesquisar as informações críticas que estejam no swap rode:
```
strings /dev/sdaX > surpresa
grep senha surpresa
less surpresa
```

### Crie um arquivo que será usado para o swap:
```
fallocate -l 3G /swapfile
```
Se `fallocate` não estiver instalado ou tiver algum problema pode usar se preferir:
```
dd if=/dev/zero of=/swapfile bs=1M count=3072
```

### Somente o usuário **root** deve poder escrever e ler o arquivo de swap.   
Para definir as permissões corretas:
```
chmod 600 /swapfile
```

## Criptografando o swap

### Colocando uma entrada no /etc/crypttab.
Com essa entrada, a cada boot será preparada uma área criptografada de swap no arquivo /swapfile.

```
# <target name> <source device> <key file>      <options>
cryptoswap         /swapfile       /dev/urandom    swap,cipher=aes-xts-plain64,size=256,hash=sha1
```

### Requisitos para usar criptografia Luks
Instalar cryptsetup-run
```
apt install cryptsetup-run
```

### Para preparar o swap criptografado
```
cryptdisks_start cryptoswap
```

### Para habilitar o swap use o comando:
```
swapon /dev/mapper/cryptoswap
```

### Para verificar se o swap está ativo pode usar:
```
swapon --summary
Filename				Type		Size	Used	Priority
/dev/dm-0                              	partition	3145724	0	-3

swapon --show
NAME      TYPE      SIZE USED PRIO
/dev/dm-0 partition   3G   0B   -2

free
              total        used        free      shared  buff/cache   available
Mem:         988076      136428      570404       12800      281244      693784
Swap:       3145724           0     3145724

```

## Colocando a entrada do swap
Caso queira que essa área de swap seja persistente.  
É necessário editar o arquivo `/etc/fstab` e colocar a entrada do swap.
```
/dev/mapper/cryptoswap  none        swap    sw            0       0
```

### Desabilitar o RESUME
Como a criptografia muda a cada boot temos que desabilitar o "resume".
```
echo "RESUME=none" >/etc/initramfs-tools/conf.d/resume

update-initramfs -u -k all
```

## Boot não prossegue
Em alguns casos, não sei ainda quais, depois de habilitar o swap criptografado o boot fica parado.   
Para esses casos o workaround a seguir resolve.

### Criando um gatilho para não dar problema de parar no boot
Repare que o nome do serviço varia de acordo com o nome que usou para criar o swap.
```
mkdir /etc/systemd/system/systemd-cryptsetup@cryptoswap.service.d/
```

## Criando o arquivo 90-trigger-udev.conf.
Nesse diretório crie um arquivo de configuração, por exemplo chamado: 90-trigger-udev.conf, nele o seguinte conteúdo.
```
# Run udevadm trigger after the mkswap call in the original generated
# service

[Service]
ExecStartPost=/sbin/udevadm trigger /dev/mapper/%i
```

## Conclusão
Desse modo poderá usar swap em um arquivo, sem precisar de uma partição dedicada para isso e com criptografia, para que ninguém que eventualmente tenha acesso físico a sua máquina consiga ter acesso ao conteúdo do swap.
