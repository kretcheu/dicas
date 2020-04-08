
# Placas Wi-Fi não amistosas

Neste artigo pretendo discutir de forma genérica os casos e desafios de usar Wi-Fi no GNU no Debian em particular.
Vamos começar por entender o que é preciso para funcionar e depois o que podemos fazer em cada caso.

## Para uma placa de rede Wi-Fi funcionar é preciso:

 -    O  módulo que é capaz de comunicar com o barramento no qual a placa está conectada.
     PCI ou USB (aqui não terá problemas)

 -    O módulo específico para o modelo da placa.

  -    (**M-A**) Na maioria dos casos o módulo já está instalado,
  -    (**M-B**) Em alguns casos é preciso baixar e compilar.

 -    Um firmware específico da placa.

  -    (**F-A**) Em alguns casos esse firmware está gravado na placa e não precisa ser carregado. Neste caso após instalar o Debian a placa já estará funcionando.
       No entanto em boa parte dos casos o firmware precisa ser carregado a cada vez que liga a placa.


  -    (**F-B**) Há um pacote livre com o firmware.
  -    (**F-C**) Há um pacote não-livre com o firmware.


O instalador do Debian, por padrão, não instala softwares não-livres (Viva os Softwares Livres!).
Placas não-amistosas aos softwares livres e que precisam carregar firmwares não-livres não funcionarão sem que você faça alguns procedimentos.

Sob o ponto de vista das liberdades de software, se a placa precisa de um firmware não-livre o mais adequado é não usá-la.
Pode trocar por outra ou usar um "dongle" que é uma placa que parece um pendrive e é conectada a uma porta USB.

Se não puder ou não quiser trocar a placa ainda pode colocá-la para funcionar, embora com perdas na sua liberdade.

## Análise dos casos:


* (**M-B**) O caso mais complexo vai exigir algumas habilidades para encontrar e preparar o módulo para rodar com sua versão de kernel.\
Esse é dos casos menos comuns e não vou tratar dele aqui, pois é muito variável a solução em função do modelo da placa, pode ver detalhes nesse outro artigo: [dkms.md](dkms.md)

* (**M-A**) (**F-A**) Placa funciona sem precisar fazer nenhuma intervenção.

* (**M-A**) (**F-B**) Bastará instalar o pacote do firmware do mesmo modo que instala qualquer outro disponível no Debian.

* (**M-A**) (**F-C**) Nesse caso pode escolher duas maneiras de instalar o pacote não-livre:


## Métodos de instalação


* Método (**X**) Incluir as seções contrib e non-free.

   Usando seu editor de texto preferido edite o arquivo: **/etc/apt/sources.list** e rode:

    apt update
    apt install pacote-do-firmware


Exemplo de sources.list:

    deb http://deb.debian.org/debian buster main contrib non-free
    deb http://deb.debian.org/debian buster-updates main contrib non-free
    deb http://security.debian.org buster/updates main contrib non-free


* Método (**Y**) Baixar o pacote separadamente, usando outro sistema ou caso tenha conectividade via cabo usando o mesmo.
  Depois do arquivo baixado rode:


        apt install <arquivo-do-firmware.deb
        ou
        dpkg -i <arquivo-do-firmware.deb>

## Como descobrir qual o meu caso?

Será preciso descobrir o nome e o id da placa para poder pesquisar de modo eficiente.

Rode: `lsusb -tv` caso a placa se conecte no barramento USB.\
Rode: `lspci -nn` caso a placa se conecte no barramento PCI

Se não souber, não tem problema, rode os dois comandos e nos resultados vai descobrir em qual sua placa está e também qual módulo ela usa, caso já esteja presente.

O id é um número de identificação com o formato: XXXX:XXXX como em: 168c:0032

De posse desse número pode pesquisar por "debian wiki" mais o id e em boa parte dos casos encontrará um artigo explicando os procedimentos.

Caso não encontre ou deseje um método alternativo pode descobrir qual arquivo de firmware está faltando para depois descobrir se existe um pacote com ele e qual é.

Rode: `grep firmware /var/log/syslog`

Deve obter algo semelhante a:

`firmware: failed to load rtl_nic/rtl8168g-3.fw`

Isso significa que o arquivo `rtl8168g-3.fw` não está disponível, para descobrir se há um pacote com esse arquivo acesse:

https://www.debian.org/distrib/packages

use o formulário: **"Procurar o conteúdo dos pacotes"** colocando o nome do arquivo no campo: **"Palavra-Chave"** e seleciona sua versão de Debian.

Se estiver disponível ficará sabendo o nome do pacote e usar os métodos (**X**) ou (**Y**).\
Depois de usar um dos métodos e ter conseguido instalar o pacote, tem duas opções:

- A mais feia rebootando o sistema. (**Rebootar "Jamais"!**)
- A mais elegante rodando:

    `modprobe -r <nome-do-módulo>`\
    `modprobe <nome-do-módulo>`

Obs.: Para saber o nome do módulo rode:\
`lspci -nnkd::0280` ou `lspci -nn`
e veja a linha: **Kernel driver in use:**

Então pode testar rodando: `ip link` para saber o nome que a interface recebeu e depois rodar:

`iw dev <nome-da-interface> scan`

Exemplo:

    ip link

    wlp2s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN mode DORMANT group default qlen 1000
    link/ether b0:10:41:df:66:23 brd ff:ff:ff:ff:ff:ff

    iw dev wlp1s0 scan

Se estiver tudo funcionando poderá ver as redes Wi-Fi ao seu alcance.

## Glossário:

**módulo** - Parte do kernel responsável por fazer a comunicação com um dispositivo ou acrescentar uma funcionalidade.

**firmware** - Software que roda no processador de um dispositivo e não no processador da máquina.

**barramento** - Conjunto de linhas de comunicação que permitem a interligação entre dispositivos, como a CPU, a memória e outros periféricos.


