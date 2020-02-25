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
