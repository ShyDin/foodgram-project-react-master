# Проект Foodgram

### Описание
Проект "Foodgram" – это "продуктовый помощник". На этом сервисе авторизированные пользователи могут публиковать рецепты, подписываться на публикации других пользователей, добавлять понравившиеся рецепты в список «Избранное». Для неавторизированных пользователей доступны просмотр рецептов и страниц авторов.


### Как запустить проект на боевом сервере:

Установить на сервере docker и docker-compose. Скопировать на сервер файлы docker-compose.yaml и default.conf:

```
scp docker-compose.yml <логин_на_сервере>@<IP_сервера>:/home/<логин_на_сервере>/docker-compose.yml
scp nginx.conf <логин_на_сервере>@<IP_сервера>:/home/<логин_на_сервере>/nginx.conf

```

Добавить в Secrets на Github следующие данные:

```
DB_ENGINE=django.db.backends.postgresql # указать, что проект работает с postgresql
DB_NAME=postgres # имя базы данных
POSTGRES_USER=postgres # логин для подключения к базе данных
POSTGRES_PASSWORD=postgres # пароль для подключения к БД
DB_HOST=db # название сервиса БД (контейнера) 
DB_PORT=5432 # порт для подключения к БД
DOCKER_PASSWORD= # Пароль от аккаунта на DockerHub
DOCKER_USERNAME= # Username в аккаунте на DockerHub
HOST= # IP удалённого сервера
USER= # Логин на удалённом сервере
SSH_KEY= # SSH-key компьютера, с которого будет происходить подключение к удалённому серверу
PASSPHRASE= #Если для ssh используется фраза-пароль
TELEGRAM_TO= #ID пользователя в Telegram
TELEGRAM_TOKEN= #ID бота в Telegram

```

Выполнить команды:

*   git add .
*   git commit -m "Коммит"
*   git push

После этого будут запущены процессы workflow:

*   проверка кода на соответствие стандарту PEP8 (с помощью пакета flake8) и запуск pytest
*   сборка и доставка докер-образа для контейнера web на Docker Hub
*   автоматический деплой проекта на боевой сервер
*   отправка уведомления в Telegram о том, что процесс деплоя успешно завершился

После успешного завершения процессов workflow на боевом сервере должны будут выполнены следующие команды:
```
sudo docker-compose exec web python manage.py migrate
```
sudo docker-compose exec web python manage.py collectstatic --no-input 
```
Затем необходимо будет создать суперюзера и загрузить в базу данных информацию об ингредиентах:
```
sudo docker-compose exec web python manage.py createsuperuser
```
sudo docker-compose exec web python manage.py load_data_csv --path <путь_к_файлу_внутри_контейнера> --model_name <имя_модели> --app_name <название_приложения>

sudo docker-compose exec web python manage.py load_data_csv --path static/data/ingredients.csv --model_name Ingredient --app_name recipes

```
Если при загрузке изображений получите ошибку "413 Request Entity Too Large", значит нужно увеличить лимит для загружаемых пользователем файлов. Для этого, на сервере, в загруженном ранее файле nginx.conf в словарь server добавьте строчку

client_max_body_size 20M;

где 20 - максимальный размер загружаемого файла в мегабайтах. Можете установить свой лимит, при желании.

```

### Как запустить проект локально в контейнерах:

Клонировать репозиторий и перейти в него в командной строке:

``` git@github.com:shydin/foodgram-project-react.git ``` 
``` cd foodgram-project-react ``` 

Запустить docker-compose:
```
docker-compose up
```
После окончания сборки контейнеров выполнить миграции:
```
docker-compose exec web python manage.py migrate
```
Создать суперпользователя:
```
docker-compose exec web python manage.py createsuperuser
```
Загрузить статику:
```
docker-compose exec web python manage.py collectstatic --no-input 
```

### В API доступны следующие эндпоинты:

* ```/api/users/```  Get-запрос – получение списка пользователей. POST-запрос – регистрация нового пользователя. Доступно без токена.

* ```/api/users/{id}``` GET-запрос – персональная страница пользователя с указанным id (доступно без токена).

* ```/api/ingredients/``` GET-запрос – получение списка всех ингредиентов. Подключён поиск по частичному вхождению в начале названия ингредиента. Доступно без токена. 

* ```/api/ingredients/{id}/``` GET-запрос — получение информации об ингредиенте по его id. Доступно без токена. 

* ```/api/recipes/``` GET-запрос – получение списка всех рецептов. Возможен поиск рецептов по тегам и по id автора (доступно без токена). POST-запрос – добавление нового рецепта (доступно для авторизированных пользователей).


### Автор проекта

**Антон Петухов** 


P.S.

Ребят, если не хотите возиться с workflow, а просто поскорее запустить проект, то вот упрощённая инструкция, нужные команды вы должны 

примерно понимать, возьмите их из текста выше:

1) Установите на ВМ docker и docker-compose

2) В папке infra найдите файлы docker-compose.yml и nginx.conf. В файле nginx.conf введите ip своей ВМ, для этого замените адрес на нужный

в строчке server_name 158.160.56.255; 

В файле docker-compose.yml замените названия контейнеров на ваши. У меня это image: shydin/backend:latest и image: shydin/frontend:latest.

Советую вам просто поменять shydin на свой ник и не изобретать велосипед.

После этого загрузите docker-compose.yml и nginx.conf на свою ВМ.

3) В папках backend и frontend есть докерфайлы, на их основе создайте два образа, загрузите их на докерхаб, а оттуда на свою ВМ

Имена образов должны совпадать с именами, данными вами в пункте 2.

4) После всех предыдущих шагов на своей ВМ выполните:

sudo docker-compose up -d

sudo docker-compose exec web python manage.py migrate

sudo docker-compose exec web python manage.py collectstatic --no-input 

sudo docker-compose exec web python manage.py createsuperuser

sudo docker-compose exec web python manage.py load_data_csv --path static/data/ingredients.csv --model_name Ingredient --app_name recipes

5) Теперь проект должен работать. Если встретите "413 Request Entity Too Large", то решение выше по тексту