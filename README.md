
# Git репозиторий + rbac

На ubuntu сервере настроить git окружение с RBAC системой доступа 

При коммите в ветку master содержимое ветки мастер должно обновлять в папке /var/www/master

При коммите в ветку dev содержимое ветки dev должно обновлять в папке /var/www/dev

При коммите в ветку отличную от dev или master ничего не делать

Дать доступ 3м(noob, pro, god) пользователям на любые операции в ветке dev

Дать доступ 2м(pro, god) пользователям на любые операции и в ветке master

# Expected result:
- Публичный репозиторий
- Склоненный с репозитория [https://github.com/MaksymSemenykhin/git-course-task/tree/task%234](https://github.com/MaksymSemenykhin/git-course-task/tree/task%234)
- Скрипта  для работы на ubuntu сервере
- Не использовать облачные git сервисы
- Не использовать teamcity, jenkins...
- Не использовать git rbac плагины, gitolite
- В README файле описано детали настройки и flow с этим репозиторием
- Репозиторий может содержать доп скрипты гит репозитория 
- Доступ на сервер для ключа god.pub, noob.pub, pro.pub без пароля
 
идея - завести пользователей с общим uid чтобы избежать проблем с правами на файлы, в настройках sshd  указать разные файлы authorized_keys, чтобы ))) не могли зайти под другим пользователем, и в качестве shell  указать /usr/bin/git-shell чтобы не молги сделать чтото кроме git.
от рута
````
root@sandbox:/etc# useradd -o -u `id -u team` -s /usr/bin/git-shell -d /home/teamcity god
root@sandbox:/etc# useradd -o -u `id -u team` -s /usr/bin/git-shell -d /home/teamcity pro
root@sandbox:/etc# useradd -o -u `id -u team` -s /usr/bin/git-shell -d /home/teamcity noob

mkdir -p /home/teamcity/git-shell-commands
cp /usr/share/doc/git/contrib/* /home/teamcity/git-shell-commands
chmod a+rx /home/teamcity/git-shell-commands/*

cat > /etc/ssh/ssh/sshd_config <<__EOFF

Match User god
    AuthorizedKeysFile<>.ssh/authorized_keys.god
Match User pro
    AuthorizedKeysFile<>.ssh/authorized_keys.pro
Match User noob
    AuthorizedKeysFile<>.ssh/authorized_keys.noob
__EOFF

sysremctl reload ssh.service
````
 готово. теперь удобный интерфейс для управления ключами. Тоесть еще одна репа. Далее юзером team
 ````
cd ~.

mkdir RBAC
cd RBAC
git init --bare RBAC.git
cat > RBAC.git/hooks/post-receive <<__EOFF
#!/bin/sh

DST_DIR=/home/teamcity/.ssh
GIT_DIR=/home/teamcity/RBAC/RBAC.git
GIT_USER=team
GIT_BTANCH=master

git --work-tree=\$DST_DIR --git-dir=\$GIT_DIR checkout --force \$GIT_BRANCH
__EOFF

cat > RBAC.git/hooks/pre-receive <<__EOFF
#!/bin/sh

DST_DIR=/home/teamcity/.ssh
GIT_DIR=/home/teamcity/RBAC/RBAC.git
GIT_USER=team
GIT_BTANCH=master


[ X\$USER != X\$GIT_USER ] && echo "ERROR: only user team can push here" && exit 1

__EOFF

````
на своей рабочей машине заводим репу для заливки ключей
````
cd ~/work
mkdir RBAC
git init
git remote add RBAC ssh://team@taska4.echo.dp.ua:60022/home/teamcity/RBAC/RBAC.git
````
коприруем сюда ключи и обзываем их соответственно.
````
authorized_keys.god
authorized_keys.pro
authorized_keys.noob
````
добавляю себе роль god  и пушу
````
cat ~/.ssh/authorized_keys >> authorized_keys.god
git add .
git commit -m 'ну что біло понятно о чем - например - пришел новій ГОД-ЛВЛ-СЕНИОР кодер на бейсике'
git push RBAC master
````
проверяем
````console
root@sandbox:/home/teamcity/.ssh# pwd.
/home/teamcity/.ssh
root@sandbox:/home/teamcity/.ssh# ls.
authorized_keys  authorized_keys.god  authorized_keys.noob  authorized_keys.pro  id_rsa  id_rsa.pub  known_hosts
root@sandbox:/home/teamcity/.ssh#.
````
 делаем попытку зайти 
````
angello@angello:~$ ssh god@elm.dp.ua -p 60022
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-76-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu Feb 13 10:34:31 EET 2020
................
  
Last login: Thu Feb 13 10:31:19 2020 from 93.78.36.46
Run 'help' for help, or 'exit' to leave.  Available commands:
list
git> help
Run 'help' for help, or 'exit' to leave.  Available commands:
list
git> list
taska3-1.echo.dp.ua.git
.BuildServer/system/caches/git/git-C01FC887.git
taska4.echo.dp.ua.git
deploy/taska3.git
RBAC/RBAC.git
git> pwd
unrecognized command 'pwd'
git> 
````
зайти можно, но консоль недоступна
````
angello@angello:~$ ssh pro@taska4.echo.dp.ua -p 60022
pro@taska4.echo.dp.ua's password:.
````
даже зайти нельзя (у него нет моего ключа)

логинимся под team на наш сервер создаем нашу прод репу
````
git init --bare taska4.echo.dp.ua.git
````
создаем хуки
````
cat > taska4.echo.dp.ua.git/hooks/update  <<__EOFF
#!/bin/sh

GOD=god
PRO=pro
NOOB=noob

refname="\$1"
oldrev="\$2"
newrev="\$3"

case \$refname in
    xref/head/master)
        case $USER in
            \$GOD|$PRO)
                exit 0
            ;;
            *)
                echo "ERROR: only $GOD and $PRO can commit here"
                exit 1
        esac
    ;;
    *)
        exit 0
    ;;
esac
__EOFF
cat > taska4.echo.dp.ua.git/hooks/post-recieve
#!/bin/sh

SITE=taska4.echo.dp.ua
GIT_DIR=/home/teamcity/\$SITE.git
GIT="git --git-dir=\$GIT_DIR"
WWW_DIR=/var/www/\$SITE


[ -w \$WWW_DIR ] || ( echo "ERROR: dir \$WWW_DIR must exists and be writable by you" && exit 1 )


while read oldrev newrev ref
do
    case \$ref in.
        refs/heads/master)
        [ -d /var/www/\$SITE/$newrev ] && echo "dir /var/www/\$SITE/\$newrev exists, cant deploy, exiting" && exit 1 || mkdir -p /var/www/\$SITE/$newrev
            echo "got update for master \$newrev for \$ref, deploying"
            \$GIT --work-tree=\$WWW_DIR/\$SITE/\$newrev checkout --force master
            rm $WWW_DIR/www
            ln -s \$WWW_DIR/\$SITE/\$newrev $WWW_DIR/\$SITE/www
        ;;
        refs/heads/dev)
            [ -d /var/www/\$SITE/dev ] || mkdir -p /var/www/\$SITE/dev
            echo "got update for dev \$newrev for \$ref, deploying"
            \$GIT --work-tree=\$WWW_DIR/master checkout --force master
        ;;

        *)
            echo "got update \$newrev for \$ref, exiting"
        ;;
    esac
done
````
проверяем.
создаем локально репу , правим index.php и запушим
````
cd ~/work
mkdir DZ4
git init
remote add DEPLOY ssh://god@taska4.echo.dp.ua:60022/home/teamcity/taska4.echo.dp.ua.git
mcedit index.php
git add index.php && git commit -m '2' && git push -f DEPLOY master
````
````console
angello@angello:~/work/DZ4$ git add index.php && git commit -m '2' && git push -f DEPLOY master
[master fa9ac0e] 2
 1 file changed, 1 insertion(+), 1 deletion(-)
Підрахунок об’єктів: 3, виконано.
Запис об’єктів: 100% (3/3), 267 bytes | 267.00 KiB/s, виконано.
Total 3 (delta 0), reused 0 (delta 0)
remote: got update fa9ac0eaa60a764469938dbdc57fae6e9c19cfc5 for refs/heads/master, deploying
remote: Already on 'master'
To ssh://taska4.echo.dp.ua:60022/home/teamcity/taska4.echo.dp.ua.git
   c862804..fa9ac0e  master -> master
angello@angello:~/work/DZ4$ 
````
теперь проверим под noob, предварительно скопировав свои ключи на сервер. на локальной машиине
````
cd ~/work
mkdir DZ4-noob
git init
git remote add DEPLOY ssh://noob@taska4.echo.dp.ua:60022/home/teamcity/taska4.echo.dp.ua.git
touch index.php
mcedit index.php
git add index.php && git commit -m 'init by noob'
git push -f DEPLOY master
````
````console
angello@angello:~/work/DZ4-noob$ git push -f DEPLOY master
Підрахунок об’єктів: 3, виконано.
Запис об’єктів: 100% (3/3), 241 bytes | 241.00 KiB/s, виконано.
Total 3 (delta 0), reused 0 (delta 0)
remote: only god and pro can commit here
remote: error: hook declined to update refs/heads/master
To ssh://taska4.echo.dp.ua:60022/home/teamcity/taska4.echo.dp.ua.git
 ! [remote rejected] master -> master (hook declined)
error: не вдалося надіслати деякі посилання в «ssh://noob@taska4.echo.dp.ua:60022/home/teamcity/taska4.echo.dp.ua.git»
angello@angello:~/work/DZ4-noob$ 
````
 чтобы больше так не обламываться 
````
git push -u -f DEPLOY master:dev
````
````console
angello@angello:~/work/DZ4-noob$ git push -u -f DEPLOY master:dev
Підрахунок об’єктів: 3, виконано.
Запис об’єктів: 100% (3/3), 241 bytes | 241.00 KiB/s, виконано.
Total 3 (delta 0), reused 0 (delta 0)
remote: got update 113c0119ed30885e0efcb24f5170b4c67a236986 for refs/heads/dev, deploying
remote: Switched to branch 'dev'
To ssh://taska4.echo.dp.ua:60022/home/teamcity/taska4.echo.dp.ua.git
 + 51a2154...113c011 master -> dev (forced update)
Гілка «master» відслідковує зовнішню гілку «dev» з «DEPLOY».
angello@angello:~/work/DZ4-noob$ 
````
проверить можно 

https://taska4.echo.dp.ua/

https://taska4-dev.echo.dp.ua/


