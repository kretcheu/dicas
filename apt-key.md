## Arrumando chaves de repositórios

Ao rodar `apt update` aparece a mensagem de alerta:

```
W: https://linux-libre.fsfla.org/pub/linux-libre/freesh/dists/freesh/InRelease: Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.
```

Isso ocorre porque a chave do repositório foi armazenada de um modo agora obsoleto.

Para alterar para o modo atual siga os passos:

1. Listando as chaves.
2. Exportando a chave.
3. Verificando se está ok.
4. Removendo a chave do modo obsoleto


1. Listando as chaves.

```
apt-key list
    Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
/etc/apt/trusted.gpg
--------------------

pub   4096R/545A3198 2013-09-07
Key fingerprint = F611 A908 FFA1 65C6 9958  4ED4 9D0D B31B 545A 3198
uid                  Jason Self <j@jxself.org>
uid                  Jason Self <jason@bluehome.net>
uid                  Jason Self <jself@gnu.org>
sub   4096R/0B3873EA 2013-09-07

```

2. Exportando a chave.

```
apt-key export 545A3198  | gpg --dearmour -o /etc/apt/trusted.gpg.d/freesh-keyring.gpg
```

veja que 545A3198 vem dos últimos 8 caractéres do código da chave.

Vai aparecer a seguinte mensagem:
```
    Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
```

3. Verificando se está ok.

```
apt update

    Reading package lists... Done
    Building dependency tree... Done
    Reading state information... Done
    All packages are up-to-date.
```

4. Removendo a chave do modo obsoleto

```
apt-key del 545A3198
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
OK

```

