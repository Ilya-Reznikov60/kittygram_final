# Kittygram

## Описание проекта
Kittygram - это уникальная платформа, которая позволяет вам окунуться в мир безграничной милоты и уютных пушистых созданий. Присоединяйтесь к нашей котомании и делятесь своими самыми красивыми и интересными моментами с вашими питомцами или любимыми уличными кошками. Будьте частью нашей дружной кошачьей семьи на Kittygram!

## Ссылка на развернутый проект
[Kittygram](https://kittygramwork.site/)

[![Build Status](https://github.com/Ilya-Reznikov60/kittygram_final/workflows/Main%20Kittygram%20workflow/badge.svg)](https://github.com/Ilya-Reznikov60/kittygram_final/actions/workflows/main.yml)

## Описание версий

Продакш-версия проекта предназначена для использования в рабочей среде или для публичного доступа. Она содержит стабильный и протестированный код, который готов к использованию в реальных условиях.

Обычная версия проекта может содержать новые функции, исправления ошибок или экспериментальный код, который еще не был полностью протестирован или утвержден для использования в продакшн среде. Она может быть полезна для разработчиков, которые хотят экспериментировать с новыми функциями или вносить изменения в проект.

Выбор того, какую версию использовать, зависит от ваших потребностей и целей. Если вы хотите использовать стабильный и проверенный код, рекомендуется использовать продакш-версию. Если вы хотите получить доступ к новым функциям или исправлениям ошибок, которые еще не были выпущены в продакш-версии, вы можете использовать обычную версию.

docker-compose.yml написан так, что образы билдятся при каждом запуске. Это удобно в процессе отладки, но в продакшене удобнее запускать контейнеры из готовых образов. 
Для этого создан отдельный файл конфигурации Docker Compose(docker-compose.production.yml), который будет управлять запуском контейнеров на боевом сервере. Он будет аналогичен тому, который уже есть, но в нём будут использоваться собранные образы из Docker Hub.

## Технологии

 - Python 3.9
 - Django==3.2.3
 - djangorestframework==3.12.4
 - Nginx
 - gunicorn
 - psycopg2-binary==2.9.3

## Установка 

1. Клонируйте репозиторий на свой компьютер:

    ```bash
    git clone git@github.com:Ilya-Reznikov60/kittygram_final.git
    ```
    ```bash
    cd kittygram
    ```
2. Создайте файл .env и заполните его своими данными. Перечень данных указан в корневой директории проекта в файле .env.example.


### Создание Docker-образов

1.  Замените username на ваш логин на DockerHub:

    ```bash
    cd frontend
    docker build -t username/kittygram_frontend .
    cd ../backend
    docker build -t username/kittygram_backend .
    cd ../nginx
    docker build -t username/kittygram_gateway . 
    ```

2. Загрузите образы на DockerHub:

    ```bash
    docker push username/kittygram_frontend
    docker push username/kittygram_backend
    docker push username/kittygram_gateway
    ```

### Деплой на сервере

1. Подключитесь к удаленному серверу

    ```bash
    ssh -i путь_до_файла_с_SSH_ключом/название_файла_с_SSH_ключом имя_пользователя@ip_адрес_сервера 
    ```

2. Создайте на сервере директорию kittygram через терминал

    ```bash
    mkdir kittygram
    ```

3. Установка docker compose на сервер:

    ```bash
    sudo apt update
    sudo apt install curl
    curl -fSL https://get.docker.com -o get-docker.sh
    sudo sh ./get-docker.sh
    sudo apt-get install docker-compose-plugin
    ```

4. В директорию kittygram/ скопируйте файлы docker-compose.production.yml и .env:

    ```bash
    scp -i path_to_SSH/SSH_name docker-compose.production.yml username@server_ip:/home/username/kittygram/docker-compose.production.yml
    * ath_to_SSH — путь к файлу с SSH-ключом;
    * SSH_name — имя файла с SSH-ключом (без расширения);
    * username — ваше имя пользователя на сервере;
    * server_ip — IP вашего сервера.
    ```

5. Запустите docker compose в режиме демона:

    ```bash
    sudo docker compose -f docker-compose.production.yml up -d
    ```

6. Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/:

    ```bash
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
    sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
    ```

7. На сервере в редакторе nano откройте конфиг Nginx:

    ```bash
    sudo nano /etc/nginx/sites-enabled/default
    ```

8. Измените настройки location в секции server:

    ```bash
    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:9000;
    }
    ```

9. Проверьте работоспособность конфига Nginx:

    ```bash
    sudo nginx -t
    ```
    Если ответ в терминале такой, значит, ошибок нет:
    ```bash
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful
    ```

10. Перезапускаем Nginx
    ```bash
    sudo service nginx reload
    ```

### Настройка CI/CD

1. Файл workflow уже написан. Он находится в директории

    ```bash
    kittygram/.github/workflows/main.yml
    ```

2. Для адаптации его на своем сервере добавьте секреты в GitHub Actions:

    ```bash
    DOCKER_USERNAME                # имя пользователя в DockerHub
    DOCKER_PASSWORD                # пароль пользователя в DockerHub
    HOST                           # ip_address сервера
    USER                           # имя пользователя
    SSH_KEY                        # приватный ssh-ключ (cat ~/.ssh/id_rsa)
    SSH_PASSPHRASE                 # кодовая фраза (пароль) для ssh-ключа

    TELEGRAM_TO                    # id телеграм-аккаунта (можно узнать у @userinfobot, команда /start)
    TELEGRAM_TOKEN                 # токен бота (получить токен можно у @BotFather, /token, имя бота)
    ```

## Автор
Резников Илья - [GitHub](https://github.com/Ilya-Reznikov60)
