# Como usar chave para autenticar em servidor ssh
Acessar um servidor ssh se autenticando por senha não é muito seguro, pois pode permitir que alguém por brute-force ou outro método venha descobrir a senha.\
Se o acesso for do usuário root é ainda mais perigoso, por isso por padrão o acesso do usuário root não é permitido com senha.

1. Criar um par de chaves.
2. Copiar a chave para o servidor.
3. Acessar o servidor.
4. Copiar a chave para o root.
5. Acessar como root.

### Etapa 1 (Criar o par de chaves)
Primeiro é necessário criar um par de chaves, uma pública e uma privada.\
Iremos usar um chave criptográfica para criptografar a chave privada, afim de não deixá-la vulnerável a ser copiada.
```
ssh-keygen -t rsa -b 4096 -C comentario
Generating public/private rsa key pair.
Enter file in which to save the key (/home/kretcheu/.ssh/id_rsa): <ENTER>
Enter passphrase (empty for no passphrase): <Chave criptográfica da chave privada>
Enter same passphrase again: <Repetir chave criptográfica da chave privada>
Your identification has been saved in /home/kretcheu/.ssh/id_rsa
Your public key has been saved in /home/kretcheu/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:oikdbxRJ4cunauADZLv0EmXWcuLEylQ1y/m3GX+iaQs comentario
The key's randomart image is:
+---[RSA 4096]----+
|    .oo.         |
|   ..o+.         |
|  o .++          |
| + O +.o         |
|= O = =.So       |
|.*.o * +. =      |
|.o=.+ +E o o .   |
| ooo.o  ..o o    |
|  .o.   .+.      |
+----[SHA256]-----+
```
Dois arquivos serão criados no diretório *~/.ssh/*:
```
id_rsa.pub - chave pública
id_rsa - chave privada (criptografada com a chave criptográfica fornecida>
```

### Etapa 2 (Copiar a chave para o servidor)
Iremos copiar a chave pública para o servidor.\
Você precisa ter um usuário válido no servidor, nesse exemplo o usuário chama usuario.\
Se o usuário do servidor tiver o mesmo nome que o usuário local, não é necessário digitar.\
A linha de comando será:

*ssh-copy-id usuario-no-servidor@ip-ou-nome-do-servidor*

```
ssh-copy-id usuario@192.168.100.134
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/kretcheu/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
usuario@192.168.100.134's password: <senha do usuario no servidor>

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'usuario@192.168.100.134'"
and check to make sure that only the key(s) you wanted were added.

```

### Etapa 3 (Acessar o servidor)

```
ssh usuario@192.168.100.134
Enter passphrase for key '/home/kretcheu/.ssh/id_rsa': <Chave criptográfica da chave privada>
Linux mini 5.6.2-gnu #1.0 SMP Tue Sep 27 12:35:59 EST 1983 x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.

usuario@servidor:~$
```

### Etapa 4 (Copiar a chave para o root)
Caso queira acessar diretamente como root vai precisar copiar a chave para o diretório do root.\
Naturalmente que para isso precisa ter as permissões necessárias.\

Logado como usuário regular use `su` ou `sudo` para ter as permissões de root.

Com su:
```
usuario@servidor:~$ su
Senha: <senha do usuário root no servidor>
root@servidor#

cat /home/usuario/.ssh/authorized_keys >>/root/.ssh/authorized_keys
exit
exit
```
Com sudo:
```
usuario@servidor:~$
sudo cat /home/usuario/.ssh/authorized_keys >>/root/.ssh/authorized_keys
[sudo] senha para usuario: <senha do usuário no servidor>
exit
```
Obs. Usei `cat` para preservar outras chaves que o arquivo pode conter.

### Etapa 5 (Acessar como root)

```
ssh root@192.168.100.134

Enter passphrase for key '/home/kretcheu/.ssh/id_rsa': <Chave criptográfica da chave privada>
Linux servidor 5.6.2-gnu #1.0 SMP Tue Sep 27 12:35:59 EST 1983 x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.

root@servidor#
```

### Trocar passphrase da chave privada

    ssh-keygen -f id_rsa -p


