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


