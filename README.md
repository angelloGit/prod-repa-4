
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
