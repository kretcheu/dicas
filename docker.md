# Oque é Docker ?

Docker é uma tecnologia de contêiner criado pela empresa Docker Inc.  Sendo um
elemento de isolamento usado no sistema operacional para executar a imagem de
outro sistema operacional.

#### Vantagem

* O Docker é open source e tem uma grande comunidade.

* Trabalha entre o Kernel e as aplicações

* Docker **não** é maquina virtual pois não virtualiza hardware

* Imagens compartilhada entre maquinas e pessoas resolvendo o problema de dependency hell.

* Docker hub um grande repositório de imagens Docker

#### Docker trabalha com imagem, contêiner e volume


* Imagem: Informação do OS

* Contêiner: Imagem rodando

* Volume: Maneira para compartilhar arquivos entre contêineres


#### Docker HuB

Para compartilhamento de imagem Docker **Nem todas são seguras**
Mais também a imagem oficiais que passam pelo crivo.

#### Instando Docker

* Debian:

`apt install docker.io`

* Gentoo:

`emerge app-emulation/docker`


#### Baixando imagem docker

`docker pull <repositorio do docker hub>[:tag da imagem]`

As tag podem ser visualizadas no docker hub

#### Listando imagem

`docker images` **ou** `docker image ls`

#### Iniciando contêiner

`docker run -ti [--name<name>] <image_id> /bin/bash`

`t` de tty

`i` de interativo

#### Listar Container

`docker container ls`

#### Obter terminal em contêiner que roda em segundo plano

`docker attack <id>`

As vezes esse comando é meio complicado, por que o contêiner pode estár rodando uma aplicação no terminal e tu já cai nela diretamente

Prefiro este:

`docker exec -it <id> <programa>` **no caso /bin/bash**

#### Contêiner em execução

`docker ps`

#### Renomear

`docker rename <id> <novonome>`

#### Remover contêiner

`docker rm <id>`

#### Remover imagem

`docker rmi <id>`

#### Docker Commit

É possível com um contêiner docker rodando criar uma imagem

`docker commit <container_id> <repositorio>[:tag]`

#### Movendo arquivos do host para o contêiner

`docker cp <path do host> <id-container:path>`

**Não é possivel mover arquivos entre contêiner!! Ou passa pela rede ou pelo host**

#### Remover todos os contêiner parados

`docker container prune`

#### LIMPA TUDO
`docker system prune -a`

#### Execução privilegiada

Para dar alguns recursos a mais de kernel

Passar a tag `--privileged`

#### Salvar e carregar imagem

`docker save <image_id> > nome.tar

`docker load < nome.tar`

#### Salvar e carregar contêiner

`docker export <id> > <aquivo.tar>`

`cat <arquivo.tar> | docker import -<nome>[:<tag>]`

### Docker Volumes

Docker volumes é uma forma de compartilhar uma pasta entre o host e o contaier ou até mesmo
entre host container e outro container

criando um volume:
`docker volume create <nomedovolume>`

a tag é `-v` ou seja `docker run -e <pastahost>:<container>`

### btrfs

é o sistema de arquivos recomendado para utilizar com docker por utilizar a tecnologia
[copy on write](https://pt.wikipedia.org/wiki/C%C3%B3pia_em_grava%C3%A7%C3%A3o)


### Conclusão

Conclui-se que o Docker é uma ferramenta leve fácil de usar e que resolve a maioria dos problemas
de dependencia facilitando assim o compartilhamento de ambientes de produção ou desenvolvimento.
