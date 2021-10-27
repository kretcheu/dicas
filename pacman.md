# Comandos básicos do pacman

```
pacman -Sy = sincroniza os repositórios.
pacman -Su = procura por atualização.
pacman -Syu = sincroniza os repositórios/procura por atualização.
pacman -Syy = sincroniza os repositórios.
pacman -Syyu = sincronização total/procura por atualização.
pacman -S pacote = instala um pacote.
pacman -R pacote = remove um pacote.
pacman -Rs pacote = remove o pacote junto com as dependências não usadas por outros pacotes.
pacman -Rsn pacote = remove o pacote junto com as dependências não usadas por outros pacotes e junto com os arquivos de configuração.
pacman -Ss pacote = procura por um pacote.
pacman -Sw pacote = apenas baixa o pacote e não o instala.
pacman -Si pacote = mostra informações de um pacote não instalado.
pacman -Qi pacote = mostra informações do pacote já instalado.
pacman -Se pacote = instala apenas as dependências.
pacman -Ql pacote = mostra todos os arquivos pertencentes ao pacote.
pacman -Qu = mostra os pacotes que serão atualizados.
pacman -Q = lista todos os pacotes instalados.
pacman -Qo arquivo = mostra a qual pacote aquele arquivo pertence.
pacman -Qdt = lista pacotes desnecessários, sem dependências
pacman -Rns $(pacman -Qqdt) = apaga pacotes desnecessários, sem dependências
pacman -A pacote.pkg.tar.gz = instala um pacote local.
pacman -Sc = deleta do cache todos os pacotes antigos.
pacman -Scc = limpa o cache, removendo todos os pacotes existentes no /var/cache/pacman/pkg/.
pacman-optimize = otimiza a base de dados do pacman.
pacman -Sdd = instala ignorando as dependências.
pacman -Rdd = elimina um pacote ignorando as dependências.
pacman-mirrors.conf = para gerenciar pacman.cof
pacman-mirrors -g = para gerar um novo mirrorlist
pacman -U home/user/arquivo.tar.xz = instalar pacotes baixados no pc
pacman -U http://www.site.com/arquivo.tar.xz = instalar pacotes baixados via download
pacman -Qem = lista pacotes instalados do repo AUR
pacman -Rscn = desinstala pacotes e suas dependencias e seus registros, tudo.
pacman -S pacote –noconfirm = Instala o pacote sem precisar confirmar com “yes/no ,S/N”…
pacman -Syu –ignoregroup pacote1 , pacote2… = sincroniza os repositórios/procura por atualização e ignora os grupos dos pacotes solicitados

```
