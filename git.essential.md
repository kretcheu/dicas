## Configuration

    git config user.name "My Name"
    git config user.email "myemail@domain.com"

## Initialize a git repository

    git init

## Add all files to be commited

    git add .

## Add one file to be commited

    git add -- name_of_file.ext

## Add all files inside one folder to be commited

    git add -- name_of_folder

## Commit

    git commit -m 'Commit description'

## Update local repository from a remote (origin = remote name, master = branch name)

    git pull origin master

## Send commit to a remote repository

    git push origin master

## Clone a remote repository

    git clone git@host:user/repository.git
ex: git clone git@github.com:dtelaroli/docker-rails.git

## Show origin details (origin = remote name)

    git remote show origin

## Create a branch

    git checkout branch_to_be_copied
    git checkout -b branch_name

## Merge branches

    git checkout branch_to_receive_merge
    git merge branch_to_be_merged

## Delete branch

    git checkout master
    git branch -d branch_name
    git push origin --delete branch_name

## List all branches

    git show-branch

## Undo all changes

    git checkout .

## Undo changes in specific file

    git checkout -- file_name.ext

## Undo commit definetelly

    git reset --hard hash_to_commit_or_branch

## Finding the guilty

    git blame filename.ext

## Checking steps

    git reflog

## Checking commits

    git log

## Criando um novo repositório por linha de comando

    touch README.md

    git init
    git add README.md
    git commit -m "first commit"
    git remote add origin https://libregit.org/kretcheu/english.git
    git push -u origin master

## Realizando push para um repositório existente por linha de comando

    git remote add origin https://libregit.org/kretcheu/english.git
    git push -u origin master

## If you want to remove the file from the Git repository and the filesystem

    git rm file1.txt
    git commit -m "remove file1.txt"
    git push origin master


## But if you want to remove the file only from the Git repository and not remove it from the filesystem

    git rm --cached file1.txt
    git commit -m "remove file1.txt"
    git push origin master

## Incluindo ou removendo repositório remoto

    git remote add apelido https://github.com/kretcheu/download.git
    git remote remove apelido

    git remote rename apelido novo-apelido

    git push apelido

## Incluindo repositório para acesso ssh

    git remote add debian git@salsa.debian.org:kretcheu-guest/tutoriais.git

## Incluido vários repositórios num mesmo remote

    git remote add all origin-host:path/proj.git
    git remote set-url --add all nodester-host:path/proj.git
    git remote set-url --add all duostack-host:path/proj.git

## Vendo as diferenças

    git diff

depois do add

    git diff --staged
    git diff --cached


## Atualizar um repositório fork do upstream

    git remote add upstream https://github.com/repositorio-original/teste.git
    git fetch upstream
    git rebase upstream/master
    git push origin master --force
