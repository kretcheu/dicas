# Entendendo a diferença entre *su* e *sudo*
O programa **su** serve para durante uma sessão de um usuário você trocar de usuário sem precisar encerrar a sessão.   
Na maioria das vezes se usa `su` para trocar para o usuário *root* para se fazer alguma tarefa administrativa.
```
kretcheu@mini:/root$ su
Senha:
root@mini:~#

```
Foi preciso conhecer e digitar a senha do usuário root.   
Repare que o cursor muda para um **#** indicando que "virou root" ou como costumo dizer:   
**"Agora é a super poderosa rutinha!"**   

Cuidado! Num GNU o root pode tudo, inclusive estragar seu sistema.

## Usando su e carregando as variáveis de ambiente do usuário
Para que ao alterar de usuário sejam carregadas as variáveis de ambiente desse novo usuário é preciso usar assim:
```
kretcheu@mini:/root$ su -
Senha:
root@mini:~#
```
No Debian 10 e em outras distribuições a variável de ambiente PATH é diferente para um usuário regular e o root, logo não usar o sinal `-` e não carregar as variáveis do root fará alguns programas não serem encontrados.   
Esse é um problema comum, pois antes o Debian tinha um comportamento diferente.

## Comparando su ao sudo
A primeira diferença é que usando sudo a senha requerida é a do usuário regular enquanto com su é a senha do root que é necessária.
Usar sudo tem algumas vantagens sobre su, num sistema administrado por mais de um usuário, cada um deles usa a sua própria senha e a senha de root não precisa ser compartilhada.  
Compartilhar a senha de root não é uma boa prática em termos de segurança, além do fato que não se poderá distinguir quando cada um fez suas tarefas.    

Usando su, se um usuário não for mais administrar o sistema será necessário alterar a senha de root e passar para os outros.
Usando sudo bastará alterar as configirações do sudo no arquivo `/etc/sudoers` e retirar as permissões do usuário ou removê-lo do grupo que tem as permissões.

### Comportamento padrão
Enquanto com sudo apenas um comando é executado com privilégios elevados, usar su abre um novo interpretador de comandos para o usuário root. 
Permenecer como usuário root pode ser perigoso, pois um erro pode ser desastroso.

### Flexibilidade
Com sudo é possível configurar para usuários específicos ou grupos quais programas eles poderão executar com privilégios de root.   

### Usando su com sudo
Ainda é possível usar su mesmo sem saber a senha de root, basta rodar:
```
sudo su
```
Deste modo digitando a senha do usuário regular poderá usar um interpretador de comandos como root, do mesmo modo que faria com su.
Claro, os mesmos cuidados mencionados tem que ser tomados.

### Configuração do sudo
As configurações que definem o comportamento do sudo estão no arquivo `/etc/sudoers`.   
Para detalhes consulte o manual de referência do Debian [Configuração sudo](https://www.debian.org/doc/manuals/debian-reference/ch01.pt.html#_sudo_configuration)

### Conclusão
Eu não diria que um é melhor que outro, apenas que em circunstâncias diferentes um pode ser mais adequado que o outro.






 
