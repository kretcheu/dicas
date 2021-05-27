# Assinar seu próprio kernel para funcionar no secureboot

Assinando um kernel personalizado para secure boot

As instruções são para o Debian e derivados, mas devem funcionar de maneira semelhante para outras distribuições, se estiverem usando shim e grub como gerenciador de boot.
Se sua distribuição não estiver usando o shim (por exemplo, Linux Foundation Preloader), deve haver etapas semelhantes para concluir a assinatura (por exemplo, HashTool em vez do MokUtil for LF Preloader) ou você pode instalar o shim para usá-lo.

O pacote do Debian para o shim é chamado shim-signed, mas informe-se sobre como instalá-lo corretamente, para não atrapalhar o seu gerenciador de inicialização.

Desde a atualização mais recente do GRUB2 (2.02) no Debian, o GRUB2 não carrega mais kernels não assinados, desde que o Secure Boot esteja ativado.

Portanto, você tem três opções para resolver esse problema:

Você assina o kernel você mesmo.
Você usa um kernel assinado e genérico da sua distribuição.
Você desabilita a Inicialização segura.

Como as opções dois e três não são realmente viáveis, estas são as etapas para assinar o kernel você mesmo.

Instruções adaptadas do Ubuntu Blog.
Antes de seguir, faça backup do diretório /boot/efi, para que você possa restaurar tudo. Siga estas etapas por sua conta e risco.

Crie a configuração para criar a chave de assinatura, salve como mokconfig.cnf:

```
# This definition stops the following lines failing if HOME isn't
# defined.
HOME                    = .
RANDFILE                = $ENV::HOME/.rnd 
[ req ]
distinguished_name      = req_distinguished_name
x509_extensions         = v3
string_mask             = utf8only
prompt                  = no

[ req_distinguished_name ]
countryName             = <YOURcountrycode>
stateOrProvinceName     = <YOURstate>
localityName            = <YOURcity>
0.organizationName      = <YOURorganization>
commonName              = Secure Boot Signing Key
emailAddress            = <YOURemail>

[ v3 ]
subjectKeyIdentifier    = hash
authorityKeyIdentifier  = keyid:always,issuer
basicConstraints        = critical,CA:FALSE
extendedKeyUsage        = codeSigning,1.3.6.1.4.1.311.10.3.6
nsComment               = "OpenSSL Generated Certificate"
```

Ajuste todas as peças com YOUR com as suas informações.

Crie a chave pública e privada para assinar o kernel:
```
openssl req -config ./mokconfig.cnf \
        -new -x509 -newkey rsa:2048 \
        -nodes -days 36500 -outform DER \
        -keyout "MOK.priv" \
        -out "MOK.der"
```

Converta a chave também no formato PEM (mokutil precisa de DER, sbsign precisa de PEM):

```
openssl x509 -in MOK.der -inform DER -outform PEM -out MOK.pem
```

Registre a chave da sua instalação do shim:
```
mokutil --import MOK.der
```

Você será solicitado a fornecer uma senha; basta usá-la para confirmar sua seleção de chave na próxima etapa; portanto, escolha uma.

Reinicie seu sistema.
Você encontrará uma tela azul de uma ferramenta chamada MOKManager.
Selecione "Registrar MOK" e, em seguida, "Visualizar chave".
Verifique se foi a sua chave que você criou na etapa 2.
Depois, continue o processo e digite a senha que você forneceu na etapa 4.
Continue com a inicialização do sistema.

Verifique se sua chave está registrada via:
```
mokutil --list-enrolled
```

Assine o kernel instalado (ele deve estar em /boot/vmlinuz-[KERNEL-VERSION]):
```
sbsign --key MOK.priv --cert MOK.pem /boot/vmlinuz-5.11.11-gnu --output /boot/vmlinuz-5.11.11-gnu.signed
```

Copie o initram do kernel não assinado, para que também tenhamos um initram para o assinado.
```
cp /boot/initrd.img-5.11.11-gnu{,.signed}
```
Atualize seu grub-config

```
update-grub
```

Reinicie seu sistema e selecione o kernel assinado. Se a inicialização funcionar, você poderá remover o kernel não assinado:

```
mv /boot/vmlinuz-5.11.11-gnu{.signed,}
mv /boot/initrd.img-5.11.11-gnu{.signed,}
update-grub

```
Agora seu sistema deve ser executado sob um kernel assinado e a atualização do GRUB2 funciona novamente. Se você deseja atualizar o kernel personalizado, pode assinar a nova versão facilmente seguindo as etapas acima novamente a partir da etapa sete. Assim, faça backup das teclas MOK (MOK.der, MOK.pem, MOK.priv).


No caso de receber a msg:
"Image was already signed; adding additional signature"

```
sbattach --remove /boot/vmlinuz-5.11.11-gnu
sbsign --key MOK.priv --cert MOK.pem /boot/vmlinuz-5.11.11-gnu --output /boot/vmlinuz-5.11.11-gnu.signed
```
####
Pegando o hash do linux-libre, assim dá para inscrever o hash e não precisar assinar a cada atualização.

```
    mokutil --import-hash $(pesign -P -h -i /boot/vmlinuz-4.18.0-193.el8.x86_64  | cut -f 2 -d ' ')
```

## Incluindo a chave do Jxself (Linux-Libre)

If they do, and you want to use it, you should fetch and install the key with which the kernels are signed:

wget https://jxself.org/linux-libre-mok.cer
Check that it's the right one. The fingerprint is provided below as both SHA-1 and SHA-256 because SHA-256 is more secure but the mokutil program and MOK Manager will show the SHA-1. Providing both here allows for easy comparison.

```
openssl x509 -noout -fingerprint -sha1 -inform der -in linux-libre-mok.cer
EA:6D:07:60:A3:DC:1E:8A:BF:41:F4:4A:F1:FF:D1:2E:C8:63:E5:7B

openssl x509 -noout -fingerprint -sha256 -inform der -in linux-libre-mok.cer
5A:39:E0:D2:DD:1E:EF:F4:DB:D3:0A:F4:1E:CA:72:7E:B7:E7:FC:1F:5A:4B:88:CC:CE:3B:52:0C:D9:66:76:FF
```

As long as it matches, enroll the key. Note that enrolling a key is a multistep process. mokutil is used to start the process but the change can only be confirmed at boot time. First:

```
mokutil --import linux-libre-mok.cer
```
You will be asked for a temporary password for this enrollment request. Remember this password; MOK Manager will ask you for it later.

Check that it's prepared to be enrolled:
```
mokutil --list-new
```
Then restart:

```
reboot
```

The MOK Manager screen should appear after your UEFI boot screen but before your GNU/Linux distro boots to confirm that the key should be added. Follow the on-screen instructions to finish enrolling the key.

Once completed you can check that it was enrolled:

```
mokutil --list-enrolled
```

Once the key has been enrolled you should also enable validation in the shim bootloader:

```
mokutil --enable-validation
```

Once again you will be asked for a temporary password. Make sure to remember it.

Restart again:

```
reboot
```

The MOK Manager screen should appear once again. Follow the on-screen instructions to enable validation.

As the last step, make sure that Secure Boot is enabled at the firmware level:

```
mokutil --sb-state
```

You should see:
SecureBoot enabled

