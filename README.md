# Web-Technologies

## Web-сервер
Понятие «веб-сервер» может относиться как к аппаратной начинке, так и к программному обеспечению. Или даже к обеим частям, работающим совместно.

1. Железо - это компьютер, который хранит файлы сайта (HTML-документы, CSS-стили, JavaScript-файлы, картинки и другие) и доставляет их на устройство конечного пользователя (веб-браузер и т.д.). Он подключён к сети Интернет и может быть доступен через доменное имя.
2. ПО - включает в себя несколько компонентов, которые контролируют доступ веб-пользователей к размещённым на сервере файлам, как минимум — это HTTP-сервер.

HTTP-сервер — это часть ПО, которая понимает URL-адреса (веб-адреса) и HTTP-протокол. Например, [Nginx](#).

![](/images/web-server-structure.png "Web-сервер")

---

## HTTP-сервер

Рассматривается Nginx.

### Запуск web-сервера

* Команда на запуск `дописать`
* Чтение файла конфигурации
* Получение (открывание) порта 80
* Открытие (создание) логов
* Понижение привелегий - чтобы не только root-пользователь имел доступ
* Запуск обработчиков дочерних процессов/потоков
* Готов к обработке запросов

### Файлы web-сервера

* Конфиг `etc/nginx/nginx.conf`  
обычно конфиг большой, поэтому включает в себя несколько файлов командой `include`
* Init-скрипт `тут должен быть указан путь`
* PID-файл `/var/run/nginx.pid` - id процесса сервера
* Error-лог `/var/log/nginx/error.log`
* Access-лог `/var/log/nginx/access.log`

Веб-серверы имеют модульную архитектуру, т.е. можно добавлять доп модули для доп функциональности. Есть ядро и к нему добавляется доп функционал.

### Секции и директивы в конфиге

**virtual host** (виртульный хост) - секция конфига web-сервера, отвечающая за обслуживание определенных доменов.  
**location** - секция конфига, отвечающая за обслуживание определенной группы URL.

* http - конфигурация для HTTP-сервера
* server - конфигурация домена (вирт. хоста)
* server_name - имена доменов
* location - см. выше
* root, alias - откуда нужно брать файлы
* error_log - лог ошибок сервера
* access_log - лог запросов

### Обработка сетевых соединений

**Проблема:** блокирующий ввод-вывод  
**Решение:**
* множество потоков (multithreading) - более трудный, но менее затратный
* множество процессов (prefork) - проще, но затратнее
* комбинированный подход
* ИЛИ использовать неблокирующий ввод-вывод - мультиплексирование

**Gunicorn** - prefork-сервер (т.е. запускает процессы на всех ядрах CPU), вводится для интеграции с Python. Используется для запуска бизнес-логики, т.е. как application-сервер.  
**Nginx** - асинхронный сервер, очень быстрый. Отдает статику, ждет ответы от Python-кода (т.е. бизнес-логики), обслуживает много запросов.



## Архитекура Frontend-Backend

**Frontend** - веб-сервер (например, nginx).  
**Backend** - сервер приложения (где выполняется бизнес-логика).

![](/images/front-back-architecture.PNG "Frontend-Backend")

### Задачи Frontend (web) сервера

* отдача статических документов
* проксирование (reverse proxy)
    * frontend (медленно) читает запрос от клиента
    * frontend (быстро) передает запрос свободному backend
    * backend генерирует запрашиваемую страницу
    * backend (быстро) возвращает ответ frontend-серверу
    * frontend (медленно) возвращает ответ клиенту
* балансировка нагрузки
* кеширование
* сборка SSI
* авторизация, SSL, нарезка картинок, gzip

*Backend занят минимальное время. Это позволяет бизнес-логике обрабатывать больше запросов на динамическое генерирование. А frontend в это время обрабатывет клиентские запросы.*

### Настройка проксирования в nginx

```
proxy_set_header Host      $proxy_host;  # заменить заголовок Host значением из проксируемого хоста (www.partner.com)
proxy_set_header X-Real-IP $remote_addr;  # чтобы при передаче запроса на backend сохранить ip-адрес клиента, а не frontend-сервера
location / {
    proxy_pass http://backend;  # proxy_pass - передать запрос по upstream
}
location /partner/ {
    proxy_pass http://www.partner.com;  # proxy_pass - передать запрос по URL
}
location ~ \.\w\w\w?\w?$ {
    root /www/static;  # нужно отдавать с диска из этой директории
}
```

**upstream** - группа backend-серверов, работающих под общим именем.

Настройка upstream в nginx:
```
upstream backend {
    server back1.example.com:8080 weight=1 max_fails=3;  # weight - вес сервера для балансировки нагрузки, max_fails - количество неответов от backend-сервера, после которого тот отбрасывается
    server back2.example.com:8080 weight=2;
    server unix:/tmp/backend.sock;  # unix-сокет
    server backup1.example.com:8080 backup;  # включается, когда остальные сервера не могут обслужить запрос
    server backup2.example.com:8080 backup;
}
```


## Application (backend) сервер

Роль app-сервера заключается в исполнении бизнес-логики приложения и генерации динамических документов.

На каждый HTTP-запрос app-сервер *запускает некоторый обработчик* в приложении. Это может быть функция, класс или программа, в зависимости от технологии.

*app-сервер занимается преобразованием HTTP-запроса в вызов обработчика приложения.*

**WSGI** - протокол запуска приложения (для Python).

### WSGI

WSGI - протокол вызова функции обработчика из app-сервера. Сам app-сервер при этом может выполняться в отдельном процессе или совпадать с web-сервером.  
Как правило, при использовании этого протокола в качестве app-сервера выступает отдельный легковесный процесс.

![](/images/wsgi.PNG "Развертывание WSGI")

### Gunicorn

**Gunicorn** - WSGI-сервер для использования в UNIX-системах.  
Переводит запросы, полученные от Nginx, в формат, который может обрабатывать ваше веб-приложение, и обеспечивает выполнение кода при необходимости.

В основном, его работа состоит из:
* Запуск пула рабочих процессов / потоков (выполнение вашего кода!)
* Переводит запросы, поступающие от Nginx, для совместимости с WSGI
* Переведит ответы WSGI вашего приложения в правильные ответы HTTP
* На самом деле вызывает код Python, когда приходит запрос
* Gunicorn может общаться с различными веб-серверами

Что Gunicorn не может сделать для вас:
* Не может работать с клиентами
* Не может поддерживать работу по SSL (без обработки https)
* Не может обеспечить работу полноценного веб-сервера, как например Nginx