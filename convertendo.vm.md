## Convertendo VMs para usar no KVM

Uma VM, do inglês Virtual Machine, máquina virtual muitas vezes é distribuída em formatos diferentes, para cada tipo de virtualizador ou hipervisor.

Alguns fornecedores oferecem o que é chamado applience, uma VM pronta com seu produto.
Em várias circunstâncias é oferecido num formato diferente do que é o padrão para o KVM.

Vamos pegar um exemplo do Zabbix, que fornece para vários hipervisores e tentar converter ou preparar para usar com KVM independente do formato que tenha sido distribuído.


https://www.zabbix.com/download_appliance

Usei esse site para baixar os vários formatos como se fosse o único para convertê-lo para usar no KVM.

Nesse caso é oferecido um arquivo compactado e empacotado de cada formato.

Para Vários hipervisores no formato ovf

 zabbix_appliance-5.2.3-ovf.tar.gz

Para Microsoft Hypervisor

 zabbix_appliance-5.2.3-vhd.zip

Para VMWare

 zabbix_appliance-5.2.3-vmx.tar.gz

Baixei os arquivos e o primeiro passo será descompactar/desempacotar de modo que tenhamos os formatos respectivos de cada hypervisor.


### Formato ovf

Rodamos:

```
tar -xvf zabbix_appliance-5.2.3-ovf.tar.gz

zabbix_appliance-5.2.3-ovf/
zabbix_appliance-5.2.3-ovf/zabbix_appliance-5.2.3.ovf
zabbix_appliance-5.2.3-ovf/zabbix_appliance-5.2.3-disk001.vmdk

```

Foi criado um diretório com 2 arquivos: ovf e vmdk.
o arquivo ovf contém as definições da VM, como memória, dispositivos, etc.
o arquivo vmdk é o disco virtual e é o que vamos trabalhar.

Há várias formas diferentes de poder trabalhar com esse disco virtual, vou optar por convertê-lo para o formato qcow2 padrão no KVM.

```
qemu-img convert -O qcow2 zabbix_appliance-5.2.3-disk001.vmdk zabbix-ovf.qcow2

```
Com o disco virtual convertido basta criar uma nova VM e selecionar o disco.



### Formato vmx (VMware)

Rodamos:

```
tar -xvf zabbix_appliance-5.2.3-vmx.tar.gz

zabbix_appliance-5.2.3-vmx/
zabbix_appliance-5.2.3-vmx/zabbix_appliance-5.2.3.vmx
zabbix_appliance-5.2.3-vmx/zabbix_appliance-5.2.3-disk1.vmdk

```

Foi criado um diretório com 2 arquivos: vmx e vmdk.
o arquivo vmx contém as definições da VM, como memória, dispositivos, etc.
o arquivo vmdk é o disco virtual e é o que vamos trabalhar.

Há várias formas diferentes de poder trabalhar com esse disco virtual, vou optar por convertê-lo para o formato qcow2 padrão no KVM.

```
qemu-img convert -O qcow2 zabbix_appliance-5.2.3-disk1.vmdk zabbix-ovf.qcow2

```
Com o disco virtual convertido basta criar uma nova VM e selecionar o disco.


### Formato vhd (MS Hypervisor)

Rodamos:

```
unzip zabbix_appliance-5.2.3-vhd.zip

zabbix_appliance-5.2.3-vhd
zabbix_appliance-5.2.3-vhd/zabbix_appliance-5.2.3.vhd

```

Foi criado um diretório com 1 arquivo vhd.
o arquivo vhd é o disco virtual e é o que vamos trabalhar.

Há várias formas diferentes de poder trabalhar com esse disco virtual, vou optar por convertê-lo para o formato qcow2 padrão no KVM.

```
qemu-img convert -O qcow2 zabbix_appliance-5.2.3.vhd zabbix-vhd.qcow2

```
Com o disco virtual convertido basta criar uma nova VM e selecionar o disco.


### Observações

Algumas versões de formato vhd e vmdk podem funcionar diretamente como disco virtual do KVM, porém preferi usar o formato padrão qcow2.


