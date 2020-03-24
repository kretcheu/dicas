# Em Construção...

# Instalando um Debian ThinClient
Esse tutorial é dedicado a quem tem uma máquina com poucos recursos e quer usar como cliente de uma máquina com mais recursos na mesma rede.

Para a instalação do sistema básico veja: [pen.bootavel.md](pen.bootavel.md)


### Pacotes do servidor X na máquina Cliente.
```
apt install xserver-xorg-input-mouse xserver-xorg-input-kbd xserver-xorg-video-dummy xserver-xorg-input-void xserver-xorg-input-libinput
```

### Pacotes específicos placa de vídeo
Instalando pacote específico da sua placa de vídeo.

Os mais comuns são:
- xserver-xorg-video-intel (intel)
- xserver-xorg-video-nouveau (nvidia)
- xserver-xorg-video-openchrome (via)
- xserver-xorg-video-radeon (amd)
- xserver-xorg-video-vesa (generic display driver)

Para instalar:
```
apt install xserver-xorg-video-radeon
```

O mais básico:
```
apt install xserver-xorg-video-vesa
```

Para funcionar com as placas mais comuns:
```
apt install xserver-xorg-video-all
```

Em casos particulares:
```
apt search xserver-xorg-video
```

### Criando um serviçco
Para o ambiente gráfico iniciar automaticamente depois do boot, vamos criar um serviço.

Edite o arquivo */etc/systemd/system/xdmcp.service* com o conteúdo:

```
[Unit]
Description=Xorg server at display client XDMCP
Requires=NetworkManager.service
After=systemd-logind.service

[Service]
Type=simple
SuccessExitStatus=0 1
Restart=always

ExecStart=/usr/bin/Xorg -query 192.168.100.134

[Install]
Alias=display-manager.service
WantedBy=graphical.target
```
Obs. altere a linha substituindo o IP do seu servidor:
```
ExecStart=/usr/bin/Xorg -query 192.168.100.134
```

### Habilitando o serviço
```
systemctl enable xdmcp
```

### Reduzindo...
Dicas de como reduzir uma instalação do Debian
<https://wiki.debian.org/ReduceDebian>

## No servidor
Preparando a máquina que será o servidor.

Edite o arquivo: */etc/lightdm/lightdm.conf* descomente a linha: enable=true
```
[XDMCPServer]
enabled=true
```
Reinicie o serviço:
```
systemctl restart lightdm.service
```

