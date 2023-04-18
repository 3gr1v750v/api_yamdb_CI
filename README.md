# Проект YaMDb (Docker + CI)

## Описание

Проект создан в рамках учебного курса Яндекс.Практикум.

- Версия проекта без Docker контейнера: https://github.com/EugeniGrivtsov/api_yamdb

- Версия проекта, с разворачивание через Docker-compose контейнеры: https://github.com/EugeniGrivtsov/api_yamdb_docker

Проект YaMDb собирает отзывы пользователей на произведения. Сами произведения
в YaMDb не хранятся, здесь нельзя посмотреть фильм или послушать музыку.
Произведения делятся на категории, такие как «Книги», «Фильмы», «Музыка».
Произведению может быть присвоен жанр из списка предустановленных
(например, «Сказка», «Рок» или «Артхаус»).
Список категорий и жанров может быть расширен администратором.

Аутентифицированные пользователи могут оставлять к произведениям текстовые
отзывы и выставлять произведениям оценку в диапазоне от одного до десяти
(целое число); из пользовательских оценок формируется усреднённая оценка
произведения — рейтинг (целое число). На одно произведение пользователь
может оставить только один отзыв.

Аутентифицированные пользователи также могут оставлять комментарии к отзывам.

### Технологии

- Python 3.7
- Django 3.2
- Django Rest Framework 3.12.4
- Simple JWT 4.8
- SQLite3
- Docker-compose 3.8
- PostgreSQL 13.0-alpine 
- Nginx 1.21.3-alpine 
- Gunicorn 20.0.4
- Ubuntu 22.04 + Docker Compose V2

### Управление пользователями через API

- Регистрация пользователя администратором проекта;
- Самостоятельная регистрация пользователей;
- Передача подтверждающего кода пользователю по электронной почте;
- Присваивание JWT токена пользователю для аутентификации;
- Изменение информации о пользователе;
- Назначение ролей пользователей для управления ресурсами проекта;

### Ресурсы проекта

- Ресурс `auth`: аутентификация.
- Ресурс `users`: пользователи.
- Ресурс `titles`: произведения, к которым пишут отзывы (определённый фильм, книга или песенка).
- Ресурс `categories`: категории (типы) произведений («Фильмы», «Книги», «Музыка»). Одно произведение может быть привязано только к одной категории.
- Ресурс `genres`: жанры произведений. Одно произведение может быть привязано к нескольким жанрам.
- Ресурс `reviews`: отзывы на произведения. Отзыв привязан к определённому произведению.
- Ресурс `comments`: комментарии к отзывам. Комментарий привязан к определённому отзыву.

## Документация
Подробное описание ресурсов доступно в документации после запуска проекта по адресу `http://127.0.0.1:8000/redoc/`.

В документации указаны эндпоинты (адреса, по которым можно сделать запрос), разрешённые типы запросов, права доступа и дополнительные параметры (паджинация, поиск, фильтрация итд.), когда это необходимо.

### Примеры запросов

- Регистрация пользователя:
```
POST /api/v1/auth/signup/
```
- Получение данных своей учетной записи:
```
GET /api/v1/users/me/
```
- Добавление новой категории:
```
POST /api/v1/categories/
```
- Удаление жанра:
```
DELETE /api/v1/genres/{slug}
```
- Частичное обновление информации о произведении:
```
PATCH /api/v1/titles/{titles_id}
```
- Получение списка всех отзывов:
```
GET /api/v1/titles/{title_id}/reviews/
```
- Добавление комментария к отзыву:
```
POST /api/v1/titles/{title_id}/reviews/{review_id}/comments/
```

## Как запустить проект:

### Настройка VM (Ubuntu 22.04) для Docker Compose V2:
Документация: https://docs.docker.com/compose/install/linux/

- Остановите работу nginx, если он у вас установлен и запущен

```sudo systemctl stop nginx```

- Установите последние обновления на виртуальную машину

```sudo apt update```

```sudo apt upgrade -y```

- Установите Docker

```sudo apt install docker.io```

- Установите Docker Compose V2

```sudo mkdir -p /usr/local/lib/docker/cli-plugins```

```sudo curl -SL https://github.com/docker/compose/releases/download/v2.17.2/docker-compose-linux-x86_64 -o /usr/local/lib/docker/cli-plugins/docker-compose```

```sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose```

### Подготовка для запуска проекта на VM:
- Скопируйте файлы docker-compose.yaml и nginx/default.conf из вашего проекта на сервер в home/<ваш_username>/docker-compose.yaml и home/<ваш_username>/nginx/default.conf соответственно. Если вы копируете файлы этого проекта, не забудьте поменять конфигурацию параметра 'image' в docker-compose.yaml.

```scp my_file username@host:<путь-на-сервере>```

### Подготовка "Actions secrets and variables":
- В разделе 'Settings' вашего репозитория, пройдите в раздел 'Secrets and Variables' - 'Actions' и создайте следующие 'repository secrets'

```
DOCKER_PASSWORD = <dockerhub password>
DOCKER_USERNAME = <dockerhub username>
HOST = <VM public IP>
USER = <VM login>
PASSPHRASE = <VM password>
SECRET_KEY = <Django secret key>
SSH_KEY = <SSH key of the machine that has access to VM> - можно получить командой 'cat ~/.ssh/id_rsa', скопировать всё, включая начало и конец ответа
TELEGRAM_TO = <your Telegram ID> - @userinfobot
TELEGRAM_TOKEN = <your Telegram bot Token> - @BotFather
DB_ENGINE = django.db.backends.postgresql
DB_NAME = <database name>
POSTGRES_USER = <user name>
POSTGRES_PASSWORD = <password>
DB_HOST = <data base host> eg: db
DB_PORT = <database port> eg: 5432
```

### Github Actions CI:
![workflow bagde](https://github.com/EugeniGrivtsov/yamdb_final/actions/workflows/yamdb_workflow.yml/badge.svg)

Запуск workflow осуществляется тригером 'push' в 'master'
- Проверка lint по PEP8
- Запуск текстов
- Создание образа докера (Image) для ./api_yamdb
- Сохранение образа докера (Image) в репозитории docker hub
- Подключение к серверу (виртуальная машина на ubuntu 22.04)
- Остановка работы текущих контейнеров на сервере, очищение временных образов
- Загрузка нового образа на сервер из репозитория docker hub
- Настройка .env файла на сервере
- Разворачивание контейнеров на сервере
- Осуществление миграций и сбора статики
- Оповещение об успешном завешении workflow через Telegram бота

## Автор
**Гривцов Евгений** - [https://github.com/EugeniGrivtsov](https://github.com/EugeniGrivtsov)
