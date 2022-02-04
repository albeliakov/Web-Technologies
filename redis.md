# Установка и настройка Redis + RQ

Ссылки:  
1) [Руководство по настройке в локалке](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-18-04-ru)
2) [Дополнение по настройке удаленного подключения](https://linuxize.com/post/how-to-install-and-configure-redis-on-debian-10/)
3) [Про виды доступа - смена режима или установка пароля](https://russianblogs.com/article/7577824509/)


## Установка и настройка
Описано по [ссылке](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-18-04-ru)

## Если осуществляется удаленный доступ
Для приема удаленных подключений Redis'ом, нужно закомментировать строку с *bind*:
```
# bind 127.0.0.1 ::1
```

`src/redis-cli -h IpServerWithRedis` - подключиться к Redis, если Redis находится на другом сервере, то необходимо указать его IP


## Рекомендации
1. Лучше завести отдельного пользователя под Redis.
2. Лучше ограничить доступ к серверу с Redis по IP и паролю.

## RQ

Если два раздельных серверах:  
1) на котором app и Redis
2) на котором воркеры

*На обоих серверах должен быть файл tasks.py, в котором описаны задачи для воркеров.*

На 1 сервере задачи устанавливаются в очередь, на 2 - исполняются воркерами.

На 2 сервере:  
* Создается директория воркера

`mkdir dir_name`

`cd dir_name`

`python3 -m venv env`

`. env/bin/activate`

`pip3 install rq`

* Загрузить файл tasks.py в созданную директорию

* Проверить, что есть подключение к 1 серверу [ссылка](https://stackoverflow.com/questions/60259702/remote-workers-on-multiple-servers-with-python-rq-redis)

`rq worker --url redis://:[your_redis_password]@[your_server_IP_address]`

Один воркер выполняется в одном процессе. Для запуска нескольких воркеров нужно создать юнит-файл [ссылка](https://python-rq.org/patterns/systemd/). Пример:

```
[Unit]
Description=RQ Worker Number %i
After=network.target

[Service]
Type=simple
WorkingDirectory=/home/debian/rq-test
Environment=LANG=en_US.UTF-8
Environment=LC_ALL=en_US.UTF-8
Environment=LC_LANG=en_US.UTF-8
ExecStart=/home/debian/rq-test/env/bin/rq worker -c config
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID
PrivateTmp=true
Restart=always

[Install]
WantedBy=multi-user.target
```

*config* выше - файл config.py c настройками воркера [подробнее](https://python-rq.org/docs/workers/)


`sudo journalctl -u rqworker@xx.service` - можно использовать, чтобы узнать подробную информации о статусе *rqworker@xx.service* [ссылка](https://unix.stackexchange.com/questions/225401/how-to-see-full-log-from-systemctl-status-service
)

`sudo systemctl daemon-reload` - для перезагрузки демона

`sudo systemctl` - получить информацию по всем службам

`sudo systemctl stop rqworker@xx.service` - остановить

`sudo systemctl disable rqworker@xx.service` - выключить

[Применение во Flask](https://blog.miguelgrinberg.com/post/running-a-flask-application-as-a-service-with-systemd)

[Получить результаты из rq](https://python-rq.org/docs/results/)
