## Camera virtual no obs

Este tutorial visa permitir que o obs-studio funcione como uma camera virtual.
Para isso vamos compilar o módulo v4l2loopback, carregar esse módulo, instalar o plugin obs-v4l2sink e colocá-lo para funcionar.

1. Compilar módulo v4l2loopback
2. Instalar plugin
3. Carregar módulo
4. Executar plugin
5. Acessar o site de vídeo conferência

### Etapa 1 Compilar módulo v4l2loopback

Para compilar o módulo, é necessário instalar o pacote v4l2loopback-dkms assim como os recursos de compilação e os cabeçalhos das versões de kernel para os quais queira compilar.

```
apt install linux-headers-amd64
apt install v4l2loopback-dkms
```

### Etapa 2 Instalar plugin

Para instalar o plugin, você pode compilar a partir dos fontes ou baixar um arquivo .deb com o plugin já previamente compilado.

Para compilar o plugin, acesse: https://github.com/CatxFish/obs-v4l2sink e siga as orientações do README.md

Nesse tutorial vamos optar pela segunda opção, vamos baixar o arquivo deb, extrair o plugin e copiá-lo para o diretório de plugins do obs-studio.
A versão do pacote deb disponível é para a arquiteura 64 bits, se usar 32 bits terá que compilar você mesmo o plugin.

Para baixar o pacote deb pré compilado:
```
wget https://github.com/CatxFish/obs-v4l2sink/releases/download/0.1.0/obs-v4l2sink.deb
```

Depois de feito o download do arquivo deb vamos extrair o conteúdo para um diretório temporário. *
```
mkdir /tmp/plugin
dpkg -x obs-v4l2sink.deb /tmp/plugin
```

Em seguida vamos copiar o plugin para o diretório de plugins do obs-studio.
```
cp /tmp/plugin/usr/lib/obs-plugins/v4l2sink.so /usr/lib/x86_64-linux-gnu/obs-plugins/
```

Depois de copiar pode apagar o diretório temporário:
```
rm /tmp/plugin -r
```

* Você poderia instalar o pacote deb com dpkg, mas esse pacote não segue os padrões do Debian corretamente e o plugin será colocado num diretório não padrão e não vai funcionar.

### Etapa 3 Carregar o módulo

Para carregar o módulo use:
```
modprobe v4l2loopback devices=1 video_nr=20 card_label="v4l2loopback" exclusive_caps=1
```

### Etapa 4 Executar o plugin

Rode o obs-studio e no menu ferramentas ou "tools" clique sobre o "v4l2sink". Na janela que se abre verifique se o dispositivo de vídeo é o mesmo do carregamento do módulo, nesse exemplo: /dev/video20. Clique no botão "start".

### Etapa 5 Acessar o site de vídeo conferência

Com o plugin rodando, ao abrir o navegador nos sites de vídeo conferência como Jitsi, google meet e outros poderá selecionar a camera virtual v4l2loopback.


### Para carregar o módulo sem precisar passar os parâmetros toda vez.

Edite o arquivo /etc/modprobe.d/v4l2loopback.conf
Incluindo o conteúdo:
```
options v4l2loopback devices=1 video_nr=20 card_label="v4l2loopbak" exclusive_caps=1
```

A partir daí para carregar o módulo digite:
```
modprobe v4l2loopback
```

### Para carregar o módulo sempre que der boot

Edte o arquivo: /etc/modules-load.d/modules.conf
inclua a linha:
```
v4l2loopback
```



