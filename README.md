# Kittygram — социальная сеть для обмена фотографиями любимых котов.

## О проекте

Проект реализует веб сайт kittigram. Kittygram - это социальная сеть, для любителей кошек. Пользователи могут добавлять фотографии своих котов и смотреть на котов, добавленных другими пользователями. В проекте настроены Github Actions для автотестов и развертывания на сервер.

Используемые технологии:

- ![Django](https://img.shields.io/badge/django-%23092E20.svg?style=for-the-badge&logo=django&logoColor=white)
- ![DjangoREST](https://img.shields.io/badge/DJANGO-REST-ff1709?style=for-the-badge&logo=django&logoColor=white&color=ff1709&labelColor=gray)
- ![React](https://img.shields.io/badge/react-%2320232a.svg?style=for-the-badge&logo=react&logoColor=%2361DAFB)
- ![Nginx](https://img.shields.io/badge/nginx-%23009639.svg?style=for-the-badge&logo=nginx&logoColor=white)
- ![Postgres](https://img.shields.io/badge/postgres-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white)
- ![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)

Установка и развертвывание проекта

Примечание: Все примеры указаны для Linux

Склонируйте репозиторий на свой компьютер:
```python
git clone git@github.com:feym4n-git/kittygram.git
```
Создайте файл .env и заполните его своими данными. Все необходимые переменные перечислены в файле .env.example, находящемся в корневой директории проекта.
Создание Docker-образов
Замените YOUR_USERNAME на свой логин на DockerHub:

```python
cd frontend
docker build -t YOUR_USERNAME/kittygram_frontend .
cd ../backend
docker build -t YOUR_USERNAME/kittygram_backend .
cd ../nginx
docker build -t YOUR_USERNAME/kittygram_gateway . 
```
Загрузите образы на DockerHub:

```
docker push YOUR_USERNAME/kittygram_frontend
docker push YOUR_USERNAME/kittygram_backend
docker push YOUR_USERNAME/kittygram_gateway
docker push YOUR_USERNAME/taski_gateway
```
Деплой на сервере
Подключитесь к удаленному серверу

```
ssh -i PATH_TO_SSH_KEY/SSH_KEY_NAME YOUR_USERNAME@SERVER_IP_ADDRESS 
```
Создайте на сервере директорию kittygram:

```
mkdir kittygram
```
Установите Docker Compose на сервер:

```
sudo apt update
sudo apt install curl
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo apt install docker-compose
```
Скопируйте файлы docker-compose.production.yml и .env в директорию kittygram/ на сервере:

```
scp -i PATH_TO_SSH_KEY/SSH_KEY_NAME docker-compose.production.yml YOUR_USERNAME@SERVER_IP_ADDRESS:/home/YOUR_USERNAME/kittygram/docker-compose.production.yml
```
Где:

```
PATH_TO_SSH_KEY - путь к файлу с вашим SSH-ключом
SSH_KEY_NAME - имя файла с вашим SSH-ключом
YOUR_USERNAME - ваше имя пользователя на сервере
SERVER_IP_ADDRESS - IP-адрес вашего сервера

```

Запустите Docker Compose в режиме демона:

```
sudo docker-compose -f /home/yc-user/kittygram/docker-compose.production.yml up -d
```

Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/:

```
sudo docker-compose -f /home/yc-user/kittygram/docker-compose.production.yml exec backend python manage.py migrate
sudo docker-compose -f /home/yc-user/kittygram/docker-compose.production.yml exec backend python manage.py collectstatic
sudo docker-compose -f /home/yc-user/kittygram/docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
```
Откройте конфигурационный файл Nginx в редакторе nano:

```
sudo nano /etc/nginx/sites-enabled/default
```
Измените настройки location в секции server:

```
location / {
    proxy_set_header Host $http_host;
    proxy_pass http://127.0.0.1:9000;
}
```
Проверьте правильность конфигурации Nginx:

```
sudo nginx -t
```
Если вы получаете следующий ответ, значит, ошибок нет:

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
Перезапустите Nginx:

```
sudo service nginx reload
```
Настройка CI/CD
Файл workflow уже написан и находится в директории:

```
kittygram/.github/workflows/main.yml
```
Для адаптации его к вашему серверу добавьте секреты в GitHub Actions:

```
DOCKER_USERNAME                # имя пользователя в DockerHub
DOCKER_PASSWORD                # пароль пользователя в DockerHub
HOST                           # IP-адрес сервера
USER                           # имя пользователя
SSH_KEY                        # содержимое приватного SSH-ключа (cat ~/.ssh/id_rsa)
SSH_PASSPHRASE                 # пароль для SSH-ключа

```