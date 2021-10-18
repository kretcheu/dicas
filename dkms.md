# DKMS
Instalando módulos e criando um pacote deb via dkms a partir dos fontes.

Para instalar os pacotes e usar o dkms vamos precisar dos privilégios da conta root.\
Você pode se tornar root digitando `su -` ou precedendo cada comando por `sudo`.

Eu prefiiro usar a conta root diretamenta, então rodo:
```
su -
Senha:
kretcheu@donaco:~ #
```

## São necessários alguns requisitos para poder compilar
Caso ainda não tenha instalado, instale o pacote **dkms** os requisitos são dependências e caso ainda não tenham sido instalados serão agora.


    apt install dkms

Como o fonte obteremos via github, o pacote git precisa ser instalado.

    apt install git

## Placa wireless rtl8723au
Veremos como exemplo para a placa de rede Wi-Fi rtl8723au.

Faremos os seguintes passos:

### Criar um diretório para nosso trabalho ficar organizado.
```
mkdir pacotes
cd pacotes
```

### Baixar os fontes do módulo
```
git clone https://github.com/lwfinger/rtl8723au.git
```

### Verificando o arquivo dkms.conf
```
cat rtl8723au/dkms.conf

PACKAGE_NAME="8723au"
PACKAGE_VERSION="0.1"
BUILD_MODULE_NAME[0]="8723au"
BUILT_MODULE_NAME[0]="8723au"
DEST_MODULE_LOCATION[0]="/kernel/drivers/net/wireless"
AUTOINSTALL="yes"
```

### Adicionando o source ao dkms
Para adicionar o módulo para ser tratado pelo dkms, usaremos `dkms add diretório-do-módulo`

Nesse caso rtl8723au.
```
dkms add rtl8723au

Creating symlink /var/lib/dkms/8723au/0.1/source ->
                 /usr/src/8723au-0.1

DKMS: add completed.

```

Será feita uma cópia dos fontes para o diretório

    /usr/src/8723au-0.1/

Para ver que o módulo já foi incluído rode:

    dkms status

    8723au, 0.1: added


### Agora para compilar

```
dkms build -m 8723au/0.1

Kernel preparation unnecessary for this kernel.  Skipping...

Building module:
cleaning build area...
make -j1 KERNELRELEASE=4.19.0-8-amd64 -C /lib/modules/4.19.0-8-amd64/build M=/var/lib/dkms/8723au/0.1/build.......................
cleaning build area...

DKMS: build completed.
```

### Verifique com dkms status
```
dkms status
8723au, 0.1, 4.19.0-8-amd64, x86_64: built
```

### Para instalar no seu sistema a versão que acaba de compilar
```
dkms install -m 8723au/0.1

8723au.ko:
Running module version sanity check.
 - Original module
   - No original module exists within this kernel
 - Installation
   - Installing to /lib/modules/4.19.0-8-amd64/updates/dkms/

depmod...

DKMS: install completed.
```

### Verifique com dkms status
```
dkms status
8723au, 0.1, 4.19.0-8-amd64, x86_64: installed
```
### Eliminando conflito com módulo antigo.
Para evitar que dois módulos sejam carregados, precisamos impedir que o rtl8xxxu seja carregado, para isso rode:
```
echo "blacklist rtl8xxxu" >/etc/modprobe.d/blacklist-rtl8xxxu.conf
```

## Essa placa precisa de firmware não-livre.
Infelizmente alguns fabricantes não são amigáveis aos Softwares Livres. Neste caso será preciso usar firmware não-livre que precisa ser carregado a cada boot.
O módulo faz isso, mas precisa dos arquivos no lugar certo.
```
mkdir -p /lib/firmware/iwlwifi
cp rtl8723au/*bin /lib/firmware/iwlwifi
```

### Caregando o módulo
Se você pretende usar na mesma máquina em que compilou, basta descarregar o módulo incompatível e carregar o módulo que fizemos e está tudo feito.
```
modprobe -r rtl8xxxu
modprobe 8723au
```
Para descarregar
```
modprobe -r 8723au
```

Se tudo correu bem, sua placa já deve estar funcionando.
Se precisa do módulo apenas na sua máquina, já completou o tutorial.

### Preparando um pacote deb
Se precisar usar em outras máquinas pode preparar um pacote deb com os fontes do módulo.  
Esse pacote deb do tipo dkms pode ser instalado em outra máquina que deverá ter os pacotes de compilação **build essential**.

Quando instalado, a cada nova versão de kernel o módulo é compilado para esse nova versão.

Para criar o pacote deb será necessário ter instalado o pacote debhelper, outros pacotes serão instalados.

```
apt install debhelper
```

Agora vamos criar o pacote deb

```
dkms mkdeb -m 8723au/0.1

Using /etc/dkms/template-dkms-mkdeb
copying template...
modifying debian/changelog...
modifying debian/compat...
modifying debian/control...
modifying debian/copyright...
modifying debian/dirs...
modifying debian/postinst...
modifying debian/prerm...
modifying debian/README.Debian...
modifying debian/rules...
copying legacy postinstall template...
Copying source tree...
Gathering binaries...Marking modules for 4.19.0-8-amd64 (x86_64) for archiving...

Creating tarball structure to specifically accomodate binaries.

Tarball location: /var/lib/dkms/8723au/0.1/tarball//8723au-0.1.dkms.tar.gz

DKMS: mktarball completed.

Copying DKMS tarball into DKMS tree...
Building binary package...dpkg-buildpackage: warning: using a gain-root-command while being root
 dpkg-source --before-build .
 fakeroot debian/rules clean
dh_clean: Compatibility levels before 9 are deprecated (level 7 in use)
 debian/rules build
 fakeroot debian/rules binary
dh_installdirs: Compatibility levels before 9 are deprecated (level 7 in use)
dh_strip: Compatibility levels before 9 are deprecated (level 7 in use)
dh_compress: Compatibility levels before 9 are deprecated (level 7 in use)
dh_installdeb: Compatibility levels before 9 are deprecated (level 7 in use)
dh_shlibdeps: Compatibility levels before 9 are deprecated (level 7 in use)
 dpkg-genbuildinfo --build=binary
 dpkg-genchanges --build=binary >../8723au-dkms_0.1_amd64.changes
dpkg-genchanges: info: binary-only upload (no source code included)
 dpkg-source --after-build .


DKMS: mkdeb completed.
Moving built files to /var/lib/dkms/8723au/0.1/deb...
Cleaning up temporary files...

```

O pacote deb ficará no diretório:
```
ls -l /var/lib/dkms/8723au/0.1/deb/
8723au-dkms_0.1_amd64.deb
```

### Pacote deb com binários apenas
Esse pacote deb binário conterá apenas os módulos compilados para a versão específica de kernel que você compilou.
Para instalar em outra máquina não é necessário que esta tenha os recursos de compilação.
No entanto, quando sair uma nova versão de kernel terá que preparar o pacote novamente.

```
dkms mkbmdeb 8723au/0.1

Using /etc/dkms/template-dkms-mkbmdeb
copying template...
modifying debian/changelog...
modifying debian/compat...
modifying debian/control...
modifying debian/copyright...
modifying debian/README.Debian...
modifying debian/rules...
copying legacy postinstall template...
Copying source tree...
Gathering binaries...Marking modules for 4.19.0-8-amd64 (x86_64) for archiving...

Creating tarball structure to specifically accomodate binaries.

Tarball location: /var/lib/dkms/8723au/0.1/tarball/8723au-0.1.dkms.tar.gz


DKMS: mktarball completed.

Copying DKMS tarball into DKMS tree...
Building binary package...dpkg-buildpackage: warning: using a gain-root-command while being root
 dpkg-source --before-build .
 fakeroot debian/rules clean
dh_clean: Compatibility levels before 9 are deprecated (level 7 in use)
 debian/rules build
 fakeroot debian/rules binary
dh_installdirs: Compatibility levels before 9 are deprecated (level 7 in use)
dh_strip: Compatibility levels before 9 are deprecated (level 7 in use)
dh_compress: Compatibility levels before 9 are deprecated (level 7 in use)
dh_installdocs: Compatibility levels before 9 are deprecated (level 7 in use)
dh_installdeb: Compatibility levels before 9 are deprecated (level 7 in use)
dh_shlibdeps: Compatibility levels before 9 are deprecated (level 7 in use)
 dpkg-genbuildinfo --build=binary
 dpkg-genchanges --build=binary >../8723au-dkms-bin_0.1_amd64.changes
dpkg-genchanges: info: binary-only upload (no source code included)
 dpkg-source --after-build .


DKMS: mkbmdeb completed.
Moving built files to /var/lib/dkms/8723au/0.1/bmdeb...
Cleaning up temporary files...
```

O pacote deb ficará no diretório:
```
ls -l /var/lib/dkms/8723au/0.1/bmdeb/
total 336
-rw-r--r-- 1 root root 341708 mar  5 21:47 8723au-modules-4.19.0-8-amd64_0.1_amd64.deb

```
Esse pacote poderá ser instalado em outra máquina sem que esta precise compilar.
Obs. A versão de kernel lá tem que ser exatamente a que usou para compilar.


### Remover
Se quiser remover o módulo instalado por `dkms install 8723au/0.1`

```
dkms uninstall 8723au/0.1

-------- Uninstall Beginning --------
Module:  8723au
Version: 0.1
Kernel:  4.19.0-8-amd64 (x86_64)
-------------------------------------

Status: Before uninstall, this module version was ACTIVE on this kernel.

8723au.ko:
 - Uninstallation
   - Deleting from: /lib/modules/4.19.0-8-amd64/updates/dkms/
 - Original module
   - No original module was found for this module on this kernel.
   - Use the dkms install command to reinstall any previous module version.

depmod...

DKMS: uninstall completed.
```

Se quiser remover do dkms
```
dkms remove 8723au/0.1
```

Os fontes continuarão no /usr/src se desejar apague com:
```
rm /usr/src/8723au-0.1 -r
```

Os módulos que tenha usado com dkms ficam no diretório: */var/lib/dkms/*

Caso queira remove-los verifique se estão e quais estão rodando:
```
ls /var/lib/dkms/
```
Se quiser remova os que não lhe forem úteis.

## Conclusão
Com dkms vimos três maneiras de trabalhar.

1. Compilar e instalar os módulos.
2. Criar um pacote deb com os fontes do módulo para ser compilado quando da instalação do pacote.
3. Criar um pacote deb apenas com os binários para poder ser instalado sem compilação.

Espero que esse artigo lhe ajude.

- Para tirar dúvidas sobre Debian, **Sobre Debian**

   - No telegram:

   - [Grupo Debian Brasil](https://t.me/debianbrasil)

   - [Grupo Debian BR](https://t.me/debianbr)


- Para me encontrar:

   - [Youtube](https://youtube.com/kretcheu2001)

   - Email: [kretcheu@gmail.com](mailto:kretcheu@gmail.com)

   - Nas mídias sociais @kretcheu

- Licença

   - Todo material aqui fornecido é licenciado sob a licença (CC BY-SA 4.0).\
<https://creativecommons.org/licenses/by-sa/4.0/>

