# Git tips by Frodo<br>
## Setup Git
List the settings
````
git config --list  // Showing system, global, and (if inside a repository) local configs
git config --list --system
git config --list --global
git config --list --local  // Specific to that single repository
````
Direct way
````
git config user.email
git config --global user.email
git config --local user.email
````
Configure Git
````
git config --global user.name "Your Name"
git config --global user.email "123456789+example@users.noreply.github.com"  # Remember to use your own private GitHub email here
git config --global init.defaultBranch main  // Change the default branch to main
git config --global color.ui auto  // Enable colorful output
git config --global pull.rebase false  // pull --rebase:  Update your local working branch with commits from the remote, but rewrite history so any local commits occur after all new commits coming from the remote, avoiding a merge commit.
ssh-keygen -t ed25519 -C <youremail>
***Copy your public SSH key to Github***
ssh -T git@github.com  // Testing your SSH connection
````
## Fix remote URLs from HTTPS to SSH for Project
````
git remote set-url origin git@github.com:username/repo.git

// Then check remote URLs again:
git remote -v
````
## Remove directory from Git but NOT local
````
git rm -r --cached myFolder
git commit . -m "Remove duplicated directory"
git push origin <your-git-branch>
````
## Changing git commit message after push (The --force-with-lease option is the safest, because it will abort if there are any upstream changes)
````
git commit --amend -m "New commit message"
git push --force-with-lease <repository> <branch> 
````
## Create a new repository on the command line
````
echo "# Labratory-1" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:Frodo-Web/Labratory-1.git
git push -u origin main
````
## Push an existing repository from the command line
````
git remote add origin git@github.com:Frodo-Web/Labratory-1.git
git branch -M main
git push -u origin main
````
## Guide
````
git log
git log -p  // разница коммитов
git log -p -2  // ограничить вывод
git log --stat // общая инфа по коммитам
git log --pretty=oneline  // Хеш + сообщение коммита
git log --pretty=format:"%h - %an, %ar : %s"  // автор + когда + сообщение
git log --since=2.weeks
git log -S function_name
git log -- path/to/file
git commit --amend

// В итоге получится единый коммит — второй коммит заменит результаты первого
$ git commit -m 'Initial commit'
$ git add forgotten_file
$ git commit --amend
//

git restore --staged CONTRIBUTING.md

// Отменить изменения:
(use "git restore <file>..." to discard changes in working directory)
	modified:   CONTRIBUTING.md

// Гит даёт такое название серверу с которого происходило клонирование, origin - имя по умолчанию
$ git remote
origin

// Адреса для чтения и записи 
$ git remote -v
origin	https://github.com/schacon/ticgit (fetch)
origin	https://github.com/schacon/ticgit (push)

$ git remote -v
bakkdoor  https://github.com/bakkdoor/grit (fetch)
bakkdoor  https://github.com/bakkdoor/grit (push)
cho45     https://github.com/cho45/grit (fetch)
cho45     https://github.com/cho45/grit (push)
defunkt   https://github.com/defunkt/grit (fetch)
defunkt   https://github.com/defunkt/grit (push)
koke      git://github.com/koke/grit.git (fetch)
koke      git://github.com/koke/grit.git (push)
origin    git@github.com:mojombo/grit.git (fetch)
origin    git@github.com:mojombo/grit.git (push)

// Добавить реп-ий
$ git remote add pb https://github.com/paulboone/ticgit
$ git remote -v
origin	https://github.com/schacon/ticgit (fetch)
origin	https://github.com/schacon/ticgit (push)
pb	https://github.com/paulboone/ticgit (fetch)
pb	https://github.com/paulboone/ticgit (push)

// Теперь можно сфетчить изменения у Пола
$ git fetch pb

// Например origin, забирает, но не сливает и не модифицирует с тем с чем работаете в данный момент
// Сливать нужно вручную когда готовы
$ git fetch [remote-name]

clone -> это как локал мастер - ремоте мастер -> pull и фетч + merge ремоте мастер с локал мастер

// Если кто то пушил до вас нужно подтянуть изменения
$ git push origin master

// Пишет бренчи, даже который уже обновились по сравнению с локал, кто куда пуллит пушит по дефолту
$ git remote show origin
  HEAD branch: master
  Remote branches:
    HCS-94249                              tracked
    JOB-4371                          tracked
    JOB-4593                          new (next fetch will store in remotes/origin)
    JOB-5460                          tracked
    JOB-6174                          tracked
    master                                 tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (local out of date)
    
Делаю так:
git fetch
git checkout JOB-4593
git log
И вижу новые коммиты на этой ветке

You don't fetch a branch, you fetch an entire remote:
git fetch origin
git merge origin/an-other-branch

// Откатиться
git reflog
git reset --hard 13a2d04d

git remote rename pb paul
git remote remove paul

// Создать ветку
git branch testing
// Переключиться на ветку
git checkout testing

$ git checkout -b iss53
То же самое что
$ git branch iss53
$ git checkout iss53

// Визуализация
$ git log --oneline --decorate --graph --all

// Операция stash берет изменённое состояние вашего рабочего каталога, то есть изменённые отслеживаемые файлы и проиндексированные изменения, и сохраняет их в хранилище незавершённых изменений, которые вы можете в любое время применить обратно.
git stash

$ git status
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   index.html

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   lib/simplegit.rb
Теперь вы хотите сменить ветку, но пока не хотите фиксировать ваши текущие наработки; поэтому вы припрячете эти изменения. Для того, чтобы припрятать изменение в выделенное для этого специальное хранилище, выполните git stash или git stash push
$ git stash
Saved working directory and index state \
  "WIP on master: 049d078 Create index file"
HEAD is now at 049d078 Create index file
(To restore them type "git stash apply")

Теперь вы можете увидеть, что рабочая копия не содержит изменений:
$ git status
# On branch master
nothing to commit, working directory clean

В данный момент вы можете легко переключать ветки и работать в любой; ваши изменения сохранены. Чтобы посмотреть список припрятанных изменений, вы можете использовать git stash list:
$ git stash list
stash@{0}: WIP on master: 049d078 Create index file
stash@{1}: WIP on master: c264051 Revert "Add file_size"
stash@{2}: WIP on master: 21d80a5 Add number to log
В данном примере, предварительно были припрятаны два изменения, поэтому теперь вам доступны три различных отложенных наработки. Вы можете применить только что припрятанные изменения, используя команду, указанную в выводе исходной команды: git stash apply. 
Если вы хотите применить одно из предыдущих припрятанных изменений, вы можете сделать это, используя его имя, вот так: git stash apply stash@{2}.

Спрятанные изменения будут применены к вашим файлам, но файлы, которые вы ранее добавляли в индекс, не будут добавлены туда снова. Для того, чтобы это было сделано, вы должны запустить git stash apply с опцией --index

// Слияние ветки hotfix с веткой master
$ git checkout -b hotfix
Switched to a new branch 'hotfix'
$ vim index.html
$ git commit -a -m 'Fix broken email address'
$ git checkout master
$ git merge hotfix
Updating f42c576..3a0874c
Fast-forward
 index.html | 2 ++
 1 file changed, 2 insertions(+)
$ git branch -d hotfix
Deleted branch hotfix (3a0874c).

$ git checkout iss53
Switched to branch "iss53"
$ vim index.html
$ git commit -a -m 'Finish the new footer [issue 53]'
[iss53 ad82d7a] Finish the new footer [issue 53]
1 file changed, 1 insertion(+)

$ git merge master

$ git checkout master
Switched to branch 'master'
$ git merge iss53
$ git branch -d iss53

// Показывает ветки которые есть в локальном репозитории (не удалённом)
$ git branch
  iss53
* master
  testing
  
// Показать слитые неслитые локальные ветки с текущей
$ git branch --merged
$ git branch --no-merged

// Переименовать локально
$ git branch --move bad-branch-name corrected-branch-name
// Запушить измение названия в удалённый
$ git push --set-upstream origin corrected-branch-name

$ git branch --all
* corrected-branch-name
  main
  remotes/origin/bad-branch-name
  remotes/origin/corrected-branch-name
  remotes/origin/main
  
// Старую нужно удалить с удалённого репа
$ git push origin --delete bad-branch-name


$ git branch --move master main
$ git push --set-upstream origin main
$ git push origin --delete master

origin/master - remote branch
master - local branch

$ git push origin serverfix

// Отправит локальную ветку serverfix в ветку awesomebranch удалённого репозитория.
$ git push origin serverfix:awesomebranch

//Это даст вам локальную ветку, в которой можно работать и которая будет начинаться там же, где и origin/serverfix.
$ git checkout -b serverfix origin/serverfix
Branch serverfix set up to track remote branch serverfix from origin.
Switched to a new branch 'serverfix'

// Если вы пытаетесь извлечь ветку, которая не существует, но существует только одна удалённая ветка с точно таким же именем, то Git автоматически создаст ветку слежения:
$ git checkout serverfix
Branch serverfix set up to track remote branch serverfix from origin.
Switched to a new branch 'serverfix'

//Теперь ваша локальная ветка sf будет автоматически получать изменения из origin/serverfix.
$ git checkout -b sf origin/serverfix

//А вот как настроить уже созданную на слежение за удалённой:
$ git branch -u origin/serverfix

//Если вы хотите посмотреть как у вас настроены ветки слежения, воспользуйтесь опцией -vv для команды git branch. 
$ git branch -vv
  iss53     7e424c3 [origin/iss53: ahead 2] Add forgotten brackets
  master    1ae2a45 [origin/master] Deploy index fix
* serverfix f8674d9 [teamone/server-fix-good: ahead 3, behind 1] This should do it
  testing   5ea463a Try something new
  
// Подтянуть актуальную инфу:
$ git fetch --all; git branch -vv


$ git checkout experiment
$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: added staged command
$ git checkout master
$ git merge experiment

// В этой команде говорится: «Переключись на ветку client, найди изменения относительно ветки server и примени их для ветки master».
$ git rebase --onto master server client
//Теперь вы можете выполнить перемотку (fast-forward) для ветки master
$ git checkout master
$ git merge client

$ git rebase master server
$ git checkout master
$ git merge server

$ git branch -d client
$ git branch -d server


Abort changes to file

git checkout -- inventories/voshod/test/hosts

Discard all changes

git clean -df       // Deletes created files
git checkout -- .   // Discard changes in files

# Move existing, uncommited work to a new branch

git switch -c <new-branch>

# Stash with untracked files too
git stash --include-untracked
git stash -u
git stash --all

# git stash commands
git stash list                  // List all pushed stashes
git stash apply stash@{n}       // Apply specific stash
git stash show -p stash@{0}     // Show the changes of specific stash (the diff)
git stash show -p -u stash@{0}  // With files created too
git stash drop stash@{0}        // Delete specific stash


# You can remove unwanted commits with git rebase. Say you included some commits from a coworker's topic branch into your topic branch, but later decide you don't want those commits.

git checkout -b tmp-branch my-topic-branch  # Use a temporary branch to be safe.
git rebase -i master  # Interactively rebase against master branch.

Remove the commits you don't want by deleting their lines
Save and quit

If the rebase wasn't successful, delete the temporary branch and try another strategy. Otherwise continue with the following instructions.

git checkout my-topic-branch
git reset --hard tmp-branch  # Overwrite your topic branch with the temp branch.
git branch -d tmp-branch  # Delete the temporary branch.

If you're pushing your topic branch to a remote, you may need to force push since the commit history has changed. If others are working on the same branch, give them a heads up.

// How to fix a screwed up branch and keep the changes saved in staging
git checkout JOB-7111-copy  // A copy of a screwed-up one
git log                          // Count how many commits we did..
git reset --soft HEAD~13         // Reset them and free up our changes
git stash
git checkout JOB-7111-fix  // Fresh branch from already updated master
git stash pop
git add .
git commit -m " "                 // Make one big commit or split up into smaller commits
git push origin JOB-7111-fix // Create a new fixed branch

// Compare two branches
git diff branch1..branch2
// Compare commits 
git log master..feature

// Delete branch remotely
git push origin -d branch-name

// Rename branch
git branch -m new-name
git branch -m old-name new-name
````
