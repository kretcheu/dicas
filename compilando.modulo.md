# Compilando um módulo separado

## copiando receita atual
```
cd /usr/src/linux-source-5.16
cp /boot/config-5.16.0-4-amd64 .config
```
## editando a receita para incluir um módulo
Editar arquivo .config

```
# CONFIG_TUN is not set
CONFIG_TUN=m
```

## Copiando os símbolos
```
cp ../linux-headers-5.16.0-4-amd64/Module.symvers .
```

## preparando
```
make scripts prepare modules_prepare
```

## comppilando módulos do diretório
```
make -C . M=drivers/net/
```
## carregando módulo recém compilado

```
insmod drivers/net/tun.ko
```
## copiando módulo compilado
```
cp drivers/net/tun.ko /lib/modules/5.16.0-4-amd64/kernel/drivers/net/
depmod
```

