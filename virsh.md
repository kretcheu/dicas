# Básico do virsh

### Listando as VMs disponíveis
```
virsh list --all
```

### Iniciando uma VM
```
virsh start --domain nome-da-vm
ou
virsh start nome-da-vm
```
Usando o primeiro formato á fácil usar a tecla TAB para completar o nome das VMs disponíveis.

### Para ver os IPs das VMs que usaram o DHCP
```
virsh net-dhcp-leases default
```

### Para desligar redes virtuais e as interfaces.
```
virsh net-destroy default
```

### Para editar uma VM
```
virsh edit nome-da-vm
```
### Para renomear uma VM
Com a VM parada.

```
virsh dumpxml nome-antigo > nome-novo.xml
vi nome-novo.xml # para alterar o nome no xml

virsh undefine nome-antigo
virsh define nome-novo.xml
virsh list --all
```

## Para editar a rede default
```
virsh net-edit default
```

### Para alterar a localização padrão dos discos virtuais.

Listanto os "pools" atuais:
```
virsh pool-list

Name                 State      Autostart
-------------------------------------------
default              active     yes
```

Desligando "Destroying" o pool:
```
virsh pool-destroy default
Pool default destroyed
```

Removendo a definição do pool
```
virsh pool-undefine default
Pool default has been undefined
```

Definindo um novo pool com nome default:
```
virsh pool-define-as --name default --type dir --target /diretorio/para/os/discos
Pool default defined
```

Configuranado para o pool iniciar automaticamente:
```
virsh pool-autostart default
Pool default marked as autostarted
```

Iniciando o pool:
```
virsh pool-start default
Pool default started
```

Verificando:
```
virsh pool-list

Name                 State      Autostart
-------------------------------------------
default              active     yes
```
A partir de agora quando criar uma nova VM o virt-manager vai informar que o novo disco será criado no diretório que indicou.

### Imprimindo os macs de todas as VMs

```
for i in `virsh list --all| awk 'NR>2 {print $2}'`; do virsh dumpxml $i | awk -F "'" '/mac address/ {print "'$i': " $2}'; done
```

### Criando modelo para usar no servidor dhcp

```
ip=1; for i in `virsh list --all| awk 'NR>2 {print $2}'`; do let "ip=ip+1"; virsh dumpxml $i | awk -F "'" '/mac address/ {print "<host mac=\x27" $2 "\x27 name=\x27" "'$i'" "\x27 ip=\x27192.168.100." "'$ip'" "\x27" "/>" }'; done

```

```
ip=1; for i in `virsh list --all| awk 'NR>2 {print $2}'`; do ((ip++)); virsh dumpxml $i | awk -F "'" '/mac address/ {print "<host mac=\x27" $2 "\x27 name=\x27" "'$i'" "\x27 ip=\x27192.168.100." "'$ip'" "\x27" "/>" }'; done
```

### Enviando sysrq

```
virsh send-key guest1 KEY_LEFTALT KEY_SYSRQ KEY_H
```
Happy Investigating!


### Verificando e alterando a rede

```
virsh net-list
virsh net-autostart --disable --network default
virsh net-list
virsh net-autostart --network default
virsh net-list
```

### Para verificar/up/down "cabo" da interface virtual da VM

- obtendo estatísticas

    virsh domifstat debian10 --interface vnet12

- verificando IP da VM

    virsh domifaddr debian10

- obtendo estado do link

    virsh domif-getlink debian10 --interface vnet12

- Desconectando "cabo"

    virsh domif-setlink debian10 --interface vnet12 down

- Reconectando "cabo"

    virsh domif-setlink debian10 --interface vnet12 up

