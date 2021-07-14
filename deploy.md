# Настройка сервера и развертывание приложения

## Настройка сервера

[Доп источник](https://www.digitalocean.com/community/tutorials/ubuntu-18-04-ru)


### Шаг 1 -  Вход в систему от root

`ssh root@your_server_ip`

Пользователь **root** является администратором в среде Linux и имеет весьма широкие права. Ввиду расширенных прав учетной записи root не рекомендуется использовать ее на постоянной основе, поскольку некоторые права, предоставляемые учетной записи **root**, дают возможность вносить деструктивные изменения, в том числе случайно.


### Шаг 2 - Создание нового пользователя

`apt update`

`adduser newuser` - создать нового пользователя

`newuser sudo` - наделить нового пользователями правами администратора (добавляется в группу sudo)


### Шаг 3 - Представление внешнего доступа для нового пользователя

`ssh-copy-id newuser@your_server_ip` - при входе копируется публичный ssh-ключ на удаленный сервер, чтобы в дальнейшеи была возможность входить без пароля

`ssh newuser@your_server_ip` - при следующем входе


### Шаг 4 - Настройка входа пользователей

`sudo nano /etc/ssh/sshd_config`

* AllowUsers **newuser** - вход по ssh есть только у newuser  
* PermitRootLogin **no** - запретить логин root'у  
* PasswordAuthentication **no** - выключить аутентификацию по паролю

`sudo service ssh restart` - перезагрузить систему ssh, чтобы обновился конфиг


### Шаг 5 - Установка nginx

`sudo apt install nginx`


### Шаг 6 - Установка Python

[Установка Python 3.9 на Debian 10](https://linuxize.com/post/how-to-install-python-3-9-on-debian-10/)

`sudo apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libsqlite3-dev libreadline-dev libffi-dev curl libbz2-dev`

`wget https://www.python.org/ftp/python/3.9.5/Python-3.9.5.tgz`  
!!! Вместо 3.9.5 можно вписать последнюю версию.

`tar -xzvf Python-3.9.5.tgz`

`cd Python-3.9.5`  
`./configure --enable-optimizations --prefix=/home/newuser/.python3.9/`

`make -j 2`  
!!! 2 - количесвто ядер на сервере, при необходимости изменить

`sudo make altinstall`

`cd ~/.python3.9/bin`  
`nano ~/.bashrc`  
* export PATH=$PATH:/home/newuser/.python3.9/bin

`source ~/.bashrc`

Теперь python доступен через команду **python3.9**

`sudo rm -rf Python-3.9.5*` - удалить ненужные директорию и архив с python


## Развертывание

### Перенос DNS-серверов

При необходимости можно перенести DNS-сервера перенести к хостеру.

### Настройка брандмауера

`sudo apt install -y ufw`  
`sudo ufw allow ssh`  
`sudo ufw allow http`  
`sudo ufw allow 443/tcp`  
`sudo ufw --force enable`  
`sudo ufw status`  

Разрешение только внешнего трафика на портах 22 (SSH), 80 (http) и 443 (HTTPS). Любые другие порты будут закрыты.

### Установка git

`sudo apt install git`

`git config --global user.name "Ivan Ivanov"`  
`git config --global user.email "ivanov@mail.ru"`

### Загрузка приложения на сервер

`mkdir myproject`  
`cd myproject`

1. git clone по http
2. git clone по ssh, но тогда необходимо сначала сгенерить ssh-ключ на сервере и занести его в удаленный репозиторий

### Настройка окружения

`python3.9 -m venv env`  
`. /env/bin/activate`  
`pip install -r requirements.txt`

Создать файл **.env** с необходимыми переменными среды: SECRET_KEY и т.п.

Нужно установить переменную среды `FLASK_APP` в точку входа приложения, чтобы команда `flask` работала, но эта переменная должна быть определена до анализа файла *.env*, поэтому её необходимо установить вручную. Чтобы избежать необходимости проделывать это каждый раз, лучше всего добавить её в нижнюю часть *~/.profile* для учетной записи ubuntu, так что она будет устанавливаться автоматически каждый раз при моем входе  
`echo "export FLASK_APP=myproject.py" >> ~/.profile`

### Настройка nginx

`sudo nano /etc/nginx/sites-enabled/myconfigname.conf`  
! рекомендуется класть конфиг в sites-available и делать ссылку на него из sites-enabled. но для простоты можно использовать sites-enabled. myconfigname.conf = например, authdemo.ru.conf

```
server {
    server_name myproject.ru;
    location / {
        include proxy_params;
        proxy_pass http://127.0.0.1:8000;  # <-- ip указан для примера
    }
}
```

`sudo nginx -s reload` - перезагрузить конфигурационный файл

### Настройка gunicorn

`pip install gunicorn`