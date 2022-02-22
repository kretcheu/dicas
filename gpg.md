# GPG #2 Mão na massa

- Pacote gpg

    apt install gpg

- criar o par de chaves

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
    gpg --recv-keys ID-CHAVE --keyserver keyserver.ubuntu.com

- atualizar chaves

    gpg --refresh-keys

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

    gpg --sign --armor ARQUIVO
    gpg --sign -a ARQUIVO
    ARQUIVO.asc

- verificar assinatura

    gpg --verify ARQUIVO.gpg
    gpg --verify ARQUIVO.asc
    gpg --output [original-filename] [signature-file]
    gpg --verify [signature-file] [original-file]

## GPG #3

- criptografar

    gpg --encrypt --sign --armor -r person@email.com name_of_file

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
