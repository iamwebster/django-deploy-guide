# Инструкция для деплоя Django-проекта на Debian 10
### Содержание
- [Инструкция для деплоя Django-проекта на Debian 10](#инструкция-для-деплоя-django-проекта-на-debian-10)
    - [Содержание](#содержание)
  - [Первичная настройка linux и обновление пакетов](#первичная-настройка-linux-и-обновление-пакетов)
    - [Настройка vim для нормальной вставки](#настройка-vim-для-нормальной-вставки)
    - [Установим python3 новой версии](#установим-python3-новой-версии)
    - [Создадим нового юзера с правами sudo](#создадим-нового-юзера-с-правами-sudo)
    - [Установим необходимые пакеты](#установим-необходимые-пакеты)
  - [Настройка postgres](#настройка-postgres)
  - [Настройка Django](#настройка-django)
    - [Склонируем проект из GitHub](#склонируем-проект-из-github)
    - [Создадим виртуальное окружение, активирем его и установим необходимые зависимости](#создадим-виртуальное-окружение-активирем-его-и-установим-необходимые-зависимости)
    - [Настроим settings.py](#настроим-settingspy)
    - [Создадим файл для переменных окружения .env](#создадим-файл-для-переменных-окружения-env)
    - [Содержимое .env](#содержимое-env)
    - [Выполнение миграций](#выполнение-миграций)
    - [Загрузка данных в БД](#загрузка-данных-в-бд)
    - [Сбор статики](#сбор-статики)
    - [Проверка работы сервера](#проверка-работы-сервера)
  - [Настройка nginx](#настройка-nginx)
  - [Настройка супервизора](#настройка-супервизора)
    - [Команды супервизора](#команды-супервизора)
    - [Если мы внесли изменения в код:](#если-мы-внесли-изменения-в-код)

## Первичная настройка linux и обновление пакетов

```bash
apt-get update
apt-get upgrade
apt-get install sudo
```

### Настройка vim для нормальной вставки
```
vim ~/.vimrc
```
Установим значение set_mouse=

### Установим python3 новой версии
```bash
sudo apt install wget software-properties-common

apt-get install build-essential

apt-get install build-essential libncursesw5-dev libreadline-gplv2-dev libssl-dev libsqlite3-dev tk-dev libc6-dev libbz2-dev libffi-dev -y 
```

```bash
wget https://www.python.org/ftp/python/3.11.3/Python-3.11.3.tgz

tar xvf Python-3.11.3.tgz
```

```bash
cd Python-3.11.3/
./configure --enable-optimizations
make install
```

```bash
cd ..
rm -rf Python-3.1.*
```

### Создадим нового юзера с правами sudo 
```bash
adduser username
usermod -aG sudo username
```

### Установим необходимые пакеты
```bash
sudo apt install postgresql nginx supervisor git
```

## Настройка postgres
```sql
sudo -u postgres psql

CREATE DATABASE db_name;
CREATE USER username WITH PASSWORD 'password';
ALTER ROLE username SET client_encoding TO 'utf8';
ALTER ROLE username SET default_transaction_isolation TO 'read committed';
ALTER ROLE username SET timezone TO 'GMT+5';
GRANT ALL PRIVILEGES ON DATABASE db_name TO username;

\q
```
## Настройка Django

### Склонируем проект из GitHub
```bash
git clone ...
```

### Создадим виртуальное окружение, активирем его и установим необходимые зависимости
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Настроим settings.py
```bash
vim settings.py
```
```python
DEBUG = False

ALLOWED_HOSTS = ['...', '127.0.0.1']
```

### Создадим файл для переменных окружения .env
```
touch .env
```

### Содержимое .env
```conf
SECRET_KEY="django-insecure-x$gpuegl8xhh1!nv!51#@&)04@_4rux^1h36fty32l221)@^_3"

POSTGRES_USER="user"
POSTGRES_PASSWORD="password"
POSTGRES_DB="db"
POSTGRES_PORT="5432"
POSTGRES_HOST="127.0.0.1"
```

### Выполнение миграций
```shell
python manage.py makemigrations
python manage.py migrate
```

### Загрузка данных в БД
```bash
python manage.py loaddata data.json
```

### Сбор статики
```bash
python manage.py collectstatic
```

### Проверка работы сервера 
```shell
gunicorn ecommerce.wsgi:application --bind ip-address
```

## Настройка nginx 
```bash
sudo vim /etc/nginx/sites-available/default 
```

```nginx
server {
    listen 80;
    server_name id-address or domain;
    access_log /var/log/nginx/example.log;

    location /static {
        alias /home/user/project/static;
        expires 30d;
    }

    location /media {
        alias /home/user/project/media;
        expires 30d;
    }

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $server_name;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
```bash
sudo service nginx restart
```
## Настройка супервизора
```bash
cd /etc/supervisor/conf.d/
sudo ln /home/sw_admin/django-ecommerce/config/ecommerce.conf
sudo update-rc.d supervisor enable
sudo service supervisor start
```

### Команды супервизора
```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl status
```

### Если мы внесли изменения в код:
```bash
sudo supervisorctl restart all
```
