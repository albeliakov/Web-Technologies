# Установка и настройка Redis + RQ

Ссылки:  
1) [Руководство по настройке в локалке](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-18-04-ru)
2) [Дополнение по настройке удаленного подключения](https://linuxize.com/post/how-to-install-and-configure-redis-on-debian-10/)
3) [Про виды доступа - смена режима или установка пароля](https://russianblogs.com/article/7577824509/)

## Установка Redis
`sudo apt update`

`sudo apt install redis-server`

## Настройка
`sudo nano /etc/redis/redis.conf` - открыть конф файл Redis

Сделать замену:
```
supervised no -> supervised systemd
```



`sudo systemctl restart redis.service` - перезапустить Redis

`sudo ufw allow proto tcp from RemoteIP to any port 6379` - добавить ip удаленного подключения (RemoteIP)

`sudo systemctl status redis` - проверить статус

Проверить доступность подключения к Redis:
* `src/redis-cli [-h IpServerWithRedis]` - подключиться к Redis, если Redis находится на другом сервере, то необходимо указать его IP
* ``

## Установка и настройка
Описано по [ссылке](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-18-04-ru)

## Если осуществляется удаленный доступ
Для приема удаленных подключений Redis'ом? нужно закомментировать строку с *bind*:
```
# bind 127.0.0.1 ::1
```

`src/redis-cli -h IpServerWithRedis` - подключиться к Redis, если Redis находится на другом сервере, то необходимо указать его IP


## Рекомендации
1. Лучше завести отдельного пользователя под Redis.
2. Лучше ограничить доступ к серверу с Redis по IP и паролю.