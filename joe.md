## Rodando programas como outro usuário

Achei um modo mais fácil.

No arquivo /etc/sudoers
incluí:
```
Cmnd_Alias JOE_CMDS = /usr/bin/irc, /usr/bin/xmpp
USUARIO ALL=(joe) NOPASSWD: JOE_CMDS
```

Daí num terminal com o seu usuário, rode:
```
xhost +
sudo -u joe irc
```

## Pode até fazer aliases:
alias joe-irc="sudo -u joe irc"
alias joe-xmpp="sudo -u joe xmpp"

Obs.:

- irc e xmpp são exemplos de nomes de programas.
Substitua pelos que deseja rodar.

- joe é um usuario aleatório sem privilégios.

- substituir USUARIO pelo nome do seu usuário.


