# Modelo mental para entender um GNU Debian.

Todos os sistemas operacionais GNU são compostos por diversas partes que pretendo descrever abaixo de forma simples, o que são, onde estão e como funcionam. O Debian não é diferente.\
Nosso objetivo é conseguir criar na sua cabeça um **mapa geral claro** sobre o sistema.\
Imagino que isso vá ajudá-lo a entender como tudo funciona e também a resolver qualquer problema que tenha que enfrentar.\
Primeiro, vou descrever algumas dessas partes imaginando a máquina desligada, nada rodando.

O sistema é composto por um conjunto de arquivos gravados num dispositivo de armazenamento como HD, CD, DVD, Pendrive, etc.

### **Boot**
Nessa categoria pensamos nos arquivos que, quando em execução, serão capazes de carregar o kernel para memória.
Esses programas são chamados de **bootloaders** e a maioria das distribuições GNU usa o GRUB (GRand Unified Bootloader), apesar de existirem vários outros como rEFInd, Syslinux e Lilo.

### **Kernel**
O Kernel mais utilizado é o Linux, mas temos o Linux-Libre que é uma versão modificada do Linux sem softwares não-livres ou outros ainda como Hurd e KFreeBSD.\
O Linux e o Linux-Livre são um arquivo chamado vmlinuz-*xyz*, que tem em torno de 10Mb e fica numa pasta chamada `/boot`.

Exemplos:

- vmlinuz-5.5.6-gnu
- vmlinuz-4.19.0-8-amd64

### **Módulos do kernel**
Nessa categoria temos muitos arquivos pequenos, são chamados módulos porque são carregados à medida  em que são necessários ao kernel para ativar recursos específicos.
 - Há os responsáveis pela comunicação do kernel com os vários dispositivos de hardware, também conhecidos como **"drivers"**.
 - Há os que implementam alguma funcionalidade específica, como entender a estrutura dos dados nos sistemas de arquivos EXT, FAT, NTFS, ou ainda entender protocolos de rede como IP.
 - Alguns de uso muito comum já foram "embutidos" no arquivo do kernel quando ele foi compilado, esses a gente chama de **"builtin"**.
Os módulos ficam numa pasta chamada `/lib/modules`. Dentro dela há sub-pastas para cada versão de kernel que temos instalado.

### **Initrd**
Existe um arquivo, **initrd**, que é uma "imagem de disco", como se fosse um HD virtual. Nele estão arquivos que o kernel vai precisar quando estiver rodando para prosseguir o boot, como por exemplo, alguns módulos que não foram embutidos.\
Este arquivo também está na mesma pasta `/boot`.

Exemplos:

- initrd.img-5.5.6-gnu
- initrd.img-4.19.0-8-amd64

### **Init**
Existem vários programas Init. Atualmente, em boa partes das distribuições GNU, é usado o **Systemd** para esse papel.\
Esse é o primeiro programa que o kernel põe para rodar. Ele é o responsável pelo que chamamos de *iniciar o sistema*, colocando para rodar tudo que foi definido para funcionar automaticamente.

### **Programas para autenticação de usuários**
Esses programas são aqueles com os quais o usuário interage digitando um nome de usuário e uma senha, para então "logar" no sistema e escolher quais os programas que deseja usar.

### **Programas servidores**
Nessa categoria, colocamos aqueles programas que são executados em segundo plano sem interação direta com o usuário, como servidores web e servidores de arquivos.\
Ficam rodando até que um outro programa comunique-se com eles.

### **Aplicativos de usuário**
Nessa categoria, entram os programas usados diretamente pelo usuário, e há uma variedade imensa deles. São os mais usados no dia a dia para fazer coisas bem básicas desde copiar arquivos numa interface em modo texto até programas como navegadores de web, editores de textos e tudo mais que usamos.

## Agora ligando a máquina.
Agora vamos por energia elétrica para as coisas acontecerem!\
Parte desse processo é feito por partes embutidas no hardware e parte pelos programas que estão nos arquivos que descrevemos.

### **BIOS**
A interface BIOS ou **"Basic Input/Output System"** está embutida no hardware, é bem antiga e nos computadores de hoje foi substituída por uma outra chamada UEFI.\
Os programas que ela possui são muito básicos e o que vamos descrever aqui é como ela acaba por conseguir carregar o boot loader.\
Quando a gente liga a máquina em poucos segundos bastante coisa acontece.\
No que estamos analisando aqui o papel principal da BIOS é carregar um programa que está nos primeiros 512 bytes do dispostivo de boot, isso mesmo, apenas 512 bytes.\
Chamamos esses 512 bytes de primeiro setor ou MBR (Master Boot Record).\
Na MBR está uma parte do bootloader também chamada de primeiro estágio do bootloader, está ali apenas para poder carregar o restante do bootloader.\
Nesse momento do processo, não se conhecem sistemas de arquivos, portanto esse programa precisa saber onde no disco estão os dados do segundo estágio do bootloader, ou seja, o bloco do disco e o tamanho dos dados. A BIOS então carrega esses dados e os põe para rodar. Temos agora o bootloader rodando e em alguns casos veremos uma tela de menu.

### **UEFI**
Essa é mais moderna e, embora também esteja embutida no hardware, tem muito mais recursos:
 - Conhece o sistema de particionamento GPT e DOS.
 - Conhece o sistema de arquivos FAT.
 - É capaz de executar programas compilados no formato PE.
 - É capaz de carregar o kernel e o initrd, dispensando a necessidade do bootloader.
 - Algumas implementam o *Secureboot*, pois podem verificar assinaturas digitais do que vai ser carregado.

Desse modo, será capaz de encontrar o arquivo do bootloader e colocá-lo para rodar.

### **Boot loader rodando...**
Seja usando a BIOS, seja usando UEFI, nesse ponto temos o **bootloader** rodando e ele é responsável por carregar para a memória o arquivo do kernel e o initrd que vimos na descrição acima.\
Depois de carregá-los, instrui o processador a executar o kernel.

### **kernel rodando...**
O kernel em execução, só deixará de ser executado quando desligarmos a máquina fisicamente ou dermos uma instrução para que ele desligue a máquina.\
Rodando, o kernel em primeiro lugar usa o initrd como um disco virtual e carrega dele alguns módulos essenciais como por exemplo o que permitirá ao kernel ter acesso ao HD da máquina.
Com todos os recursos mínimos carregados o kernel então executa o primeiro programa que genericamente chamamos de **init**.

### **Init (systemd)**
O init vai por em funcionamento as coisas mais básicas para o sistema, como montar a estrutura de diretórios, colocar dispositivos de rede para funcionar e iniciar a execução de todos os programas servidores configurados para iniciar automaticamente.\
O primeiro deles é o programa servidor de logs, pois a partir daí tudo que acontecer fica registrado em arquivos e pode ser inspecionado posteriormente em caso de problemas.

### **Serviços**
São muitos e para várias coisas, como um servidor ssh para acesso remoto, um servidor de impressão e especialmnte em máquinas desktop um servidor gráfico e um gestor de login gráfico. Daqueles que a gente vê na tela gráfica para colocar usuário e senha.

### **Login**
 - **Login texto**
Por padrão, há um programa de login que tem interface em modo texto nas máquinas em que não há interface gráfica que aparecerá na tela.\
Ao digitar usuário e senha esse programa troca informações com outros e é capaz de identificar e autenticar esse usuário, caso o par usuário e senha correspondam.\
Nesse momento, é executado o interpretador de comandos também chamado genericamente de **bash** ou **shell**, o usuário então poderá digitar **linhas de comandos** que serão interpretadas para a execução de programas aplicativos de usuários para fazer tudo o que precisa.
 - **Login gráfico**
No caso de máquinas de uso pessoal, é extremamente comum termos o ambiente gráfico e um programa de login com interface gráfica. Nele você escolhe um usuário e digita uma senha, esse programa se comunica com outros para identificar e autenticar o usuário e faz o que chamamos de abrir um sessão de algum ambiente gráfico com menus, ícones e tudo mais que você conhece bem.
A partir dessa interface, você pode executar os programas aplicativos de usuários e fazer tudo que precisa.

### **Conclusão**
Com essa descrição completa, porém simples, espero que tenha compreendido tudo que ocorre durante o boot e possa aproveitar esse conhecimento para acrescentar os detalhes que faltam para o uso do sistema GNU. Eu prefiro o Debian.

Glossário:

- [GNU](https://pt.wikipedia.org/wiki/GNU)
- [Linux-Libre](https://pt.wikipedia.org/wiki/GNU_Linux-libre)
- [Hurd](https://pt.wikipedia.org/wiki/GNU_Hurd)
- [KfreeBSD](https://wiki.debian.org/Debian_GNU/kFreeBSD)
- [EXT](https://pt.wikipedia.org/wiki/Extended_file_system)
- [FAT](https://pt.wikipedia.org/wiki/FAT32)
- [NTFS](https://pt.wikipedia.org/wiki/NTFS)
- [MBR](https://pt.wikipedia.org/wiki/Master_Boot_Record)
- [BIOS](https://pt.wikipedia.org/wiki/BIOS)
- [GPT](https://pt.wikipedia.org/wiki/Tabela_de_Parti%C3%A7%C3%A3o_GUID)
- [DOS](https://pt.wikipedia.org/wiki/Particionamento_de_disco)
- [PE](https://pt.wikipedia.org/wiki/Portable_Executable)
