
# Настройка Nginx для Laravel

В этом руководстве описаны шаги по настройке и подключению Nginx для работы с Laravel с использованием Docker.

## Шаг 1: Настройка `docker-compose.yml`

Добавьте сервис для Nginx в ваш `docker-compose.yml`:

```yaml
version: '3'

services:
  nginx:
    image: nginx:latest
    container_name: nginx_container
    volumes:
      - .:/var/www/html
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
    ports:
      - "8080:80"
    networks:
      - sne

networks:
  sne:
    driver: bridge
```

## Шаг 2: Настройка конфигурации Nginx

Создайте файл конфигурации Nginx `nginx.conf` в директории `nginx/`:

```nginx
server {
    listen 80;
    server_name localhost;

    root /var/www/html/public;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }

    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
}
```

Этот файл указывает Nginx на использование директории `public` в качестве корневого каталога для Laravel и обрабатывает PHP через FPM.

## Шаг 3: Настройка файла `.env`

Убедитесь, что в вашем `.env` файле правильно настроены параметры базы данных и других сервисов:

```bash
APP_URL=http://localhost:8080
```

## Шаг 4: Запуск сервисов

После того как вы настроили Docker и Nginx, запустите контейнеры:

```bash
docker-compose up -d
```

Теперь ваш Laravel проект будет доступен по адресу `http://localhost:8080`.

## Шаг 5: Дополнительные настройки

Вы можете добавить дополнительные настройки в конфигурационный файл Nginx, такие как настройка SSL, кэширование, и т.д.

```nginx
location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
    expires 365d;
}
```

Теперь вы готовы использовать Nginx для работы с вашим Laravel проектом через Docker.