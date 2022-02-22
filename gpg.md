# GPG #2 Mão na massa

- Pacote gpg

    apt install gpg

- criar chave

    gpg --gen-key
    gpg --full-generate-key

- listar chaves

    gpg --list-keys
    gpg --list-secret-keys

- procurar / baixar

   gpg --search-keys PESQUISA
   gpg --keyserver keyserver.ubuntu.com --search-keys kretcheu
   gpg --keyserver pgp.mit.edu --search-keys kretcheu

- baixar chaves

    gpg --recv-keys ID-CHAVE
    gpg --recv-keys ID-CHAVE --keyserver pgp.mit.edu

- remover chave

   gpg --delete-keys
   gpg --delete-secret-keys
   gpg --delete-secret-and-public-keys

- enviar servidor

    gpg --send-keys --keyserver pgp.mit.edu

- criptografar arquivo

    gpg -c arquivo.tgz

- descriptografar

    gpg -d arquivo.tgz.gpg > arquivo.tgz

- assinar arquivo

    gpg --sign ARQUIVO

    ARQUIVO.gpg

- verificar assinatura

    gpg --check-sigs EMAIL ARQUIVO.gpg
    gpg --check-sigs ID-CHAVE ARQUIVO.gpg

## GPG #3

- assinar chave

    gpg

- exportar para backup

    gpg -a --export FINGER-PRINT > chave-publica.key
    gpg -a --export-secret-keys FINGER-PRINT > chave-privada.key

    gpg --export-secret-keys 5E3A2D8ADA62D240622EC84239F17A5F5AEFBE73 > minha-chave.asc

- importar (do backup)

    gpg --list-keys
    gpg --list-public-keys

    echo "pinentry-mode loopback" >> ~/.gnupg/gpg.conf

    gpg --import chave-publica.key
    gpg --import chave-privada.key

    gpg --import chave-publica.key chave-privada.key
    gpg --list-keys
    gpg --list-secret-keys

- editar chave

    gpg --edit-key

- criar chave de revogação

- enviar chave de revogação para um servidor de chaves


## Avançado
### Artigo sobre criação de chaves

https://www.digitalneanderthal.com/post/gpg/ 
