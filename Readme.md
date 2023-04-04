# Online store
Online store with pagination and filtering of goods.

User registration, authorization via vk and github. Confirmation of e-mail using celery

Shopping cart, order payment via stripe payment system

Caching pages using redis.

Administration for adding/deleting/editing/viewing products/users/orders. 

History and status orders

## Before install
Project is configured to work with the stripe payment system

https://stripe.com/docs 

vk and google are used for OAuth 2.0 authorization 

https://vk.com/dev/access_token 

https://developers.google.com/identity/protocols/oauth2?hl=en 

# deploy on vps server

## connect to server:

    ssh root@ip_server
## creating a new user for security

    adduser name_user
 
    adding permissions

    usermod -aG sudo name_user
    usermod -a -G name_user www-data
## install PostgreSQL

    sudo apt update
    
    sudo apt install postgresql postgresql-contrib python3-venv

    sudo -u postgres psql

```
CREATE DATABASE name_db;
CREATE ROLE user_db with password 'password';
ALTER ROLE "user_db" WITH LOGIN;
GRANT ALL PRIVILEGES ON DATABASE "name_db" to user_db;
ALTER USER user_db CREATEDB;
```
## clone repository 


    mkdir for store

    cd cat for store

    git clone https://github.com/se-andrey/StoreOnDjango ./

    create venv

    python3 -m venv venv

    source venv/bin/activate

## create .env

    nano .env

```
DEBUG=False
DOMAIN_NAME=your_ip_or_domain
SECRET_KEY=django_secret_key

REDIS_HOST=ip_host_reddis
REDIS_PORT=port_reddis

DB_NAME=name_db
DB_USER=user_db
DB_PASSWORD=password
DB_HOST=localhost
DB_PORT=5432

EMAIL_HOST=smtp.mail.ru
EMAIL_PORT=2525
EMAIL_USE_TLS=True
EMAIL_USE_SSL=False
EMAIL_HOST_USER=your_email
EMAIL_HOST_PASSWORD=password_host

STRIPE_PUBLIC_KEY=stripe_public_key
STRIPE_SECRET_KEY=stripe_secret_key
STRIPE_WEBHOOK_SECRET=webhook_secret_key

GOOGLE_CLIENT_ID=123
GOOGLE_SECRET=456
```

## Install requirements
    pip install -r requirements.txt
## make migrations

    python3 manage.py migrate

## make static

    python3 manage.py collectstatic

## Install gunicorn

    cd store

    pip install gunicorn

    sudo nano /etc/systemd/system/gunicorn.socket

```
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=socket.target
```

    sudo nano /etc/systemd/system/gunicorn.service

```
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=<username>
Group=www-data
WorkingDirectory=path_to_directory_with_project
ExecStart=path_to_venv/venv/bin/gunicorn \
        --access-logfile - \
        --workers 3 \
        --bind unix:/run/gunicorn.sock \
        name_project.wsgi:application

[Install]
WantedBy=multi-user.target
```
### add host
    sudo nano /etc/hosts

in file add - 127.0.0.1 store-server


### run socket

    sudo systemctl start gunicorn.socket

    sudo systemctl enable gunicorn.socket

    sudo systemctl enable gunicorn

### check gunicorn

    sudo systemctl status gunicorn.socket

    file /run/gunicorn.sock

### to restart gunicorn (if needed)

    sudo systemctl daemon-reload

    sudo systemctl restart gunicorn 

### log gunicorn

    sudo journalctl -u gunicorn

## install nginx

    sudo apt install nginx

    sudo nano /etc/nginx/sites-available/name_project

```
server {
    listen 80;
    server_name ip_or_domain

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/user/directory_with_project/myproject;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
```
    sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled

    sudo nginx -t

    sudo systemctl restart nginx

## install redis

    sudo apt install redis-server

    sudo nano /etc/redis/redis.conf

change 'supervised no' to 'supervised systemd'

    sudo systemctl restart redis.service

## install celery

    sudo nano /etc/systemd/system/celery.service

```
[Unit]
Description=Celery Service
After=network.target

[Service]
User=user_name
Group=www-data
WorkingDirectory=path_to_project
ExecStart=path_to_project/venv/bin/celery -A store worker -l INFO

[Install]
WantedBy=multi-user.target
sudo systemctl enable celery
sudo systemctl start celery
```
## install firewall

    sudo apt install ufw

    sudo ufw app list

    sudo ufw allow OpenSSH

    sudo ufw allow 'Nginx Full'

    sudo ufw enable

    sudo ufw status

## ssl certificate 

    sudo snap install core; sudo snap refresh core

    sudo apt remove certbot

    sudo snap install --classic certbot

    sudo ln -s /snap/bin/certbot /usr/bin/certbot

    sudo certbot --nginx -d your_domain_name

## last modifications

    python3 manage.py createsuperuser

fill in the categories and products in the admin panel or fill out the fixtures

    sudo nano /etc/nginx/sites-enable/name_project
```
add after "location /static/...":
    location /media/ {
        root /home/username/dir_with_project/name_project
    }
```
    sudo systemctl restart nginx

for Oauth 2.0 to work, add vk and github site data to the admin panel

for online payment with stripe see instruction https://stripe.com/docs/webhooks/go-live  