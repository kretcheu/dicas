# Caso da NVRAM cheia!

## Histórico
Durante a instalação do Debian Buster (10.3) ao chegar no momento de gravar o GRUB dava erro.  
Com o instalador ainda rodando teclamos **ctrl-alt-f4** para ver o terminal de mensagens.  
Nele aparecia:

 **vars_set_variable: write() failed : No space left on device.** 

## Vamos a pesquisa/solução do problema

Ainda sem nos darmos conta da causa do problema.

1. Demos boot pelo Live-cd do Debian 10.3.
2. Na tela do GRUB teclamos c.  
Para rodar o shell (interpretador de comandos) do GRUB.  
Nele rodamos `ls`
com o resultado:   

```
(proc) (hd0) (hd1) (hd1,gpt3) (hd1,gpt2) (hd1,gpt1) (hd2)
```

Com esses resultados ficou claro que o sistema tinha sido instalado no (hd1).  
Rodamos:   
```
ls (hd1,gpt1)/
... Filesystem type fat, UUID ....
ls (hd1,gpt2)/
... Filesystem type ext* - Last modification ....
ls (hd1,gpt3)/
No known filesystem detected ....
```
Isso nos dá as dicas que precisamos:

- (hd1,gpt1) Foi criado o sistema de arquivos FAT para o UEFI.
- (hd1,gpt2) Que aí foi instalado o sistema.
- (hd1,gpt3) Que essa foi a partição usada para swap.


3. Ainda no shell do GRUB rodamos:
```
ls (hd1,gpt2)/boot/grub/grub.cfg
error: file '/boot/grub/grub.cfg' not found.
```
Isso nos deu a certeza que o arquivo de configuração do GRUB não foi criado na instalação.   
Vamos **sem ele!**

4. Ainda no shell do GRUB...  
Sem arquivo de configuração do GRUB vamos na "unha".  
Com ajuda da tecla **TAB** para completar os nomes, rodamos:
```
linux (hd1,gpt2)/boot/vmlinuz-4.19.0-8-amd64 root=/dev/sda2
initrd (hd1,gpt2)/boot/initrd.img-4.19.0-8-amd64
boot
```

5. Já com o Debian bootado...  
Não conceguimos logar nem com **root** nem com o usuário regular que imaginei que tivesse sido criado.  
Teclamos **ctrl-alt-f2** para ir num terminal texto logar, mas sem sucesso.  
Teclamos **ctrl-alt-del** para dar reboot e começar de novo no GRUB do live-cd.

6. Na tela do GRUB...   
Teclamos **c** para ir para o shell do GRUB.  
Já sabíamos os lugares das coisas então, repetimos apenas acrescentando **init=/bin/bash** na linha linux.  
Ficou assim:
```
linux (hd1,gpt2)/boot/vmlinuz-4.19.0-8-amd64 root=/dev/sda2 init=/bin/bash
initrd (hd1,gpt2)/boot/initrd.img-4.19.0-8-amd64
boot
```

7. Agora no terminal do Debian  
Não sabíamos a senha, mas essa forma de carregar o kernel passando **init=/bin/bash** nos dá um terminal como root (**hduhu!**).  
Então vamos definir a senha:
```
passwd
New password:
Retype new password:
passwd: Authentication token manipulation error
passwd: password unchanged
```
Foi aí que lembramos que o **/** deve ter sido montado **read-only**, então rodamos:
```
mount -o remount,rw /
```
Pronto agora o **/** está montado **read-write** e poderemos definir a senha:
```
passwd
New password:
Retype new password:
update sucessfully
```

8. Não existia nenhum usuário então criamos um:
```
adduser usuario
```
Digitamos a senha duas vezes e ok, **usuário criado**   
Agora teclados **ctrl-alt-del** e vamos para o GRUB de novo.

9. De novo no GRUB.
Teclamos **c** para ir para shell e nele de novo:
```
linux (hd1,gpt2)/boot/vmlinuz-4.19.0-8-amd64 root=/dev/sda2
initrd (hd1,gpt2)/boot/initrd.img-4.19.0-8-amd64
boot
```
Dessa fez tudo como esperado, pudemos logar com o usuário inclusive no modo gráfico.  
Abrimos um terminal e rodamos:
```
su -
```
Agora como root:
```
grub-install /dev/sda  
efivarfs_set_variable: writing to fd 6 failed: No space left on device
```
Ou seja, o mesmo que o instalador já nos havia apresentado.   
Então agora vamos ver o que acomtece rodando:
```
efibootmgr -v
```
Apareceram **20** entradas de **0000** até **0015** (é hexadecial!)   
Hum hum!! Esse deve ser o problema muitas entradas a NVRAM lotada.   
Vamos apagar umas entradas e ver no que dá:
```
efibootmgr -b 15 -B
Could not delete variable: No space left on device
```
Nada feito :(

10. Idas e vindas   
Bem, sem muito a que recorrer, bootamos no setup da BIOS, resetamos, enquanto pesquisavamos por alguma solução.   
É um erro recorrente em equipamentos **Samsung**, não é relacionado a nenhum sistema operacional em particular.   
Tem haver com a implementação UEFI deles que tem ~~alguns~~ muitos bugs.  
Entre idas e vindas, achei numa discussão de um relato de bug no Debian uma luz no fim do túnel.    
<https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=845023>

11. Nos finalmentes   
Demos boot de novo, usando o pendrive como fizemos das outras vezes.   
Como sugeria na lista do Debian rodamos:
```
mkdir /boot/efi/EFI/BOOT
cp  /boot/efi/EFI/debian/grubx64.efi /boot/efi/EFI/BOOT/BOOTX64.EFI
```
Essa é uma entrada bem padrão.   
Copiamos o **grub64.efi** com nome **BOOTX64.EFI**  
Para criar o arquivo de configuração do GRUB usamos:   
```
update-grub
Generating grub configuration file ...
Found background image: /usr/share/images/desktop-base/desktop-grub.png
Found linux image: /boot/vmlinuz-4.19.0-8-amd64
Found initrd image: /boot/initrd.img-4.19.0-8-amd64
Adding boot menu entry for EFI firmware configuration
done
```

12. Rebootando...   
Pronto tudo em ordem. :)

13. Conclusão    
Provavelmente nas várias tentativas de instalação do Debian e também de sistemas antigos e ainda aquele da tela azul,
muitas entradas da UEFI foram criadas e os limites da **NVRAM** chegaram ao fim.   
Como a implementação da UEFI está com BUGs não se consegue nem apagar nem muito menos incluir uma entrada.  
Não é uma "solução" propriamente dita, mas nosso amigo poderá desfrutar do seu Debian.   
Ficamos ainda na dúvida se uma atualização na BIOS pode resolver, mas isso vai ficar para outro dia.

14. Agora só nos resta...   

**Viva o Debian!**   
**Viva o Software Livre!**
