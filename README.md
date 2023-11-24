# Проект Taski

## _Описание_

Проект представляет собой приложение с интерфейсом для создания задач. В нем можно писать задачи, которые необходимо выполнить и отмечать выполненные.

## Настройка приложения

Клонировать репозиторий и перейти в него в командной строке:

```
git clone git@github.com:Aleksandri-A/taski-docker.git
```

Скачайте и установите curl — консольную утилиту, которая умеет скачивать файлы по команде пользователя:
```
sudo apt update
sudo apt install curl
```

Запустите скрипт:
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
```

Дополнительно к Docker установите утилиту Docker Compose:
```
sudo apt-get install docker-compose-plugin 
```

В терминале в папке с docker-compose.yml выполните команду:

```
docker compose up 
```

Перейдите в директорию, где лежит файл docker-compose.yml, и выполните миграции:

docker compose exec backend python manage.py migrate
Выполните команду сборки статики. 

```
docker compose exec backend python manage.py collectstatic
```

## Деплой: публикация проекта в Docker на сервере

Поочерёдно выполните на сервере команды для установки Docker и Docker Compose для Linux.

```
sudo apt update
sudo apt install curl
curl -fSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
sudo apt-get install docker-compose-plugin 
```

Создайте на сервере директорию taski и файл docker-compose.production.yml и скопируйте в него сожержимое из локального docker-compose.production.yml.

Создайте файл .env и внесите ваши данные.

Для запуска Docker Compose в режиме демона команду выполните эту команду на сервере в папке taski/:
```
sudo docker compose -f docker-compose.production.yml up -d 
```

Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/:
```
sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/ 
```

Если Nginx ещё не установлен на удалённый сервер, установите его:
```
sudo apt install nginx -y
```

Запустите Nginx командой:
```
sudo systemctl start nginx
```

На сервере в редакторе nano откройте конфиг Nginx и обновите настройки: 
```
nano /etc/nginx/sites-enabled/default
```

```
server {
    listen 80;
    server_name ваш_домен;
    location /api/ {
        proxy_pass http://127.0.0.1:8000;
    }
    location /admin/ {
        proxy_pass http://127.0.0.1:8000;
    }
    location / {
        proxy_pass http://127.0.0.1:8000;
    }
}
```

Чтобы убедиться, что в конфиге нет ошибок — выполните команду проверки конфигурации:
```
sudo nginx -t 
```

Перезагрузите конфиг Nginx:
```
sudo service nginx reload 
```
