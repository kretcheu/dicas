# Modelo mental para entender um GNU Debian.

Num GNU qualquer, e no Debian não é diferente temos algumas partes, vou descrever simplificadamente o que são, onde estão e como funcionam.
Nosso objetivo é que consiga criar na sua cabeça um **mapa geral claro** sobre o sistema.   
Imagino que isso vá ajudá-lo a entender como tudo funciona e também a resolver qualquer problema que tenha que enfrentar.  
    
Primeiro vou descrever algumas dessas partes imaginando a máquina desligada, nada rodando.
   
1. O sistema é composto por um conjunto de arquivos gravados num dispositivo de armazenamento como HD, CD, DVD, Pendrive, etc.

- **Boot**   
Nessa categoria pensamos nos arquivos que quando em execução serão capazes de carregar o kernel para memória.
Chamamos de **bootloaders** os programas com esse função, há vários, na maioria das distribuições GNU é usado o GRUB. 
Outros muito comuns são rEFInd, Syslinux, Lilo e por aí vai.

- **Kernel**   
O Kernel no nosso caso mais usual é o Linux, mas temos o Linux-Libre que é uma versão modificada do Linux sem softwares não-livres ou outros ainda como Hurd e KFreeBSD.
O Linux e o Linux-Livre são um arquivo chamado **vmlinuz-???**, hoje em dia tem em torno de 10Mb e estão numa pasta chamada `/boot`.   
exemplos:
      vmlinuz-5.5.6-gnu
      vmlinuz-4.19.0-8-amd64

- **Módulos do kernel**   
Nessa categoria temos muitos arquivos pequenos, são chamados módulos porque a medida da necessidade são carregados pelo kernel para trabalhar com ele.
 - Há os responsáveis pela comunicação do kernel com os vários dispositivos de hardware, também conhecidos como **"drivers"**.
 - Há os que implementam alguma funcionalidade específica como entender como as informações são armezanadas nos sistemas de arquivos como EXT, FAT, NTFS, ou ainda entender protocolos de rede como IP.
 - Alguns de uso muito comum já foram "embutidos" no arquivo do kernel quando ele foi compilado, esses a gente chama de **"builtin"**.  
Os módulos ficam numa pasta chamada `/lib/modules/` dentro dela há sub-pastas para cada versão de kernel que temos instalado.


- **Initrd**   
Existe um arquivo **initrd** que é uma "imagem de disco", como se fosse um HD virtual, nele terão arquivos que o kernel vai precisar quando estiver rodando para proceguir o boot.
Como por exemplo alguns módulos que não foram embutidos.  
Também está nessa mesma pasta `/boot`.   
exemplos:
      initrd.img-5.5.6-gnu
      initrd.img-4.19.0-8-amd64

- **Init**   
Existem vários também, atualmente em boa partes das distribuições GNU é usado o **Systemd** para esse papel.  
Esse é o primeiro programa que o kernel vai por para rodar. Ele é quem vai ser o responsável pelo que a gente chamaria de **iniciar o sistema**, colocando para rodar tudo que foi definido para funcionar automaticamente.   

- **Programas para autenticação de usuários**   
Esses programas são aqueles em que um usuário vai interagir digitando um nome de usuário e uma senha, para então "logar" no sistema e poder escolher programas para usar.
 
- **Programas servidores**   
Nessa categoria colocamos aqueles programas que são executados em segundo plano sem uma interação direta com o usuário.
Ficam rodando até que um outro programa se comunique com eles.

- **Aplicativos de usuário**   
Nessa categoria entram os programas usados diretamente pelos usuários, há uma variedade imensa aí, esses são os que a gente mais usa no dia a dia.
Desde coisas bem básicas como copiar arquivos numa interface em modo texto até programas como navegadores de web, editores de textos e tudo mais que a gente usa. 

# Agora ligando a máquina.

- BIOS / UEFI
- boot loader
- kernel
- initrd
- init (systemd)
- serviços
- login
- bash

