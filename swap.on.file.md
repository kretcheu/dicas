# Usando swap em arquivo

## Como adicionar um arquivo de swap

1. Crie um arquivo que será usado para swap:
```
sudo fallocate -l 1G /swapfile
```
Se `fallocate` não estiver instalado ou tiver algum problema pode usar se preferir:
```
 sudo dd if=/dev/zero of=/swapfile bs=1M count=1024
```

2. Somente o usuário **root** deve poder escrever e ler o arquivo de swap.   
Para definir as permissões corretas:
```
sudo chmod 600 /swapfile
```

3. Use o programa `mkswap` para preparar o arquivo como área de swap:
```
sudo mkswap /swapfile
```

4. Para habilitar o swap use o comando:
```
sudo swapon /swapfile
```

5. Para que o swap estseja sempre disponível, edite o arquivo **/etc/fstab** e acrescente a linha:
```
/swapfile none swap sw 0 0
```

6. Para verificar se o swap está ativo pode usar:
```
sudo swapon --show
NAME      TYPE  SIZE   USED PRIO
/swapfile file 1024M      0   -1
```
ou o programa `free`:
```
sudo free -h
              total        used        free      shared  buff/cache   available
Mem:          2488M        310M         83M        2.3M        246M        217M
Swap:          1.0G           0        1.0G
```


## Como ajustar o valor do **swappiness**

Swappiness é uma propriedade do kernel Linux que define com que frequência o kernel irá usar espaço de swap.
O valor do swappiness deve ser entre **0** e **100**.

Um valor baixo fará o kernel tentar evitar usar swap sempre que possível enquanto um valor alto fará o kernel usar o espaço de swap mais agressivamente.
O valor padrão do swappiness é **60**.

Você pode verificar o valor atual com o comando:
```
cat /proc/sys/vm/swappiness
60
```
O valor 60 de swappiness é adequado para a maioria dos sistemas, em servidores, pode ser necessário um valor menor.

Por exemplo, para definir o valor swappiness em 30, deve rodar:
```
sudo sysctl vm.swappiness=30
```
Para que esse parâmetro seja persistente depois de um "reboot", acrescente a seguinte linha ao arquvio **/etc/sysctl.conf**
```
vm.swappiness=30
```
O valor ideal para swappiness depende da carga do seu sistema e como a memória vem sendo usada.
Você pode ajustar esse parâmetro em pequenos incrementos para encontrar o valor ideal.

## Como remover o arquivo de swap

Se por alguma razão quiser desativar e remover o arquivo de swap, siga os passos:

1. Primeiro desative o swap:
```
sudo swapoff -v /swapfile
```

2. Remova a entrada `/swapfile none swap sw 0 0` do arquivo **/etc/fstab**.

3. Por fim remova o arquivo usando o programa rm:
```
sudo rm /swapfile
```
