
# Настройка Apache для Laravel

В этом руководстве описаны шаги по настройке и подключению Apache для работы с Laravel с использованием Docker.

## Шаг 1: Настройка `docker-compose.yml`

Добавьте сервис для Apache в ваш `docker-compose.yml`:

```yaml
version: '3'

services:
  apache:
    image: webdevops/php-apache:8.0
    container_name: apache_container
    volumes:
      - .:/var/www/html
      - ./apache/vhost.conf:/etc/apache2/sites-enabled/000-default.conf
    ports:
      - "8080:80"
    networks:
      - sne

networks:
  sne:
    driver: bridge
```

## Шаг 2: Настройка виртуального хоста Apache

Создайте файл конфигурации виртуального хоста Apache `vhost.conf` в директории `apache/`:

```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/public

    <Directory /var/www/html/public>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Этот файл указывает Apache на использование директории `public` в качестве корневого каталога для Laravel и позволяет использовать файлы `.htaccess`.

## Шаг 3: Настройка файла `.env`

Убедитесь, что в вашем `.env` файле правильно настроены параметры базы данных и других сервисов:

```bash
APP_URL=http://localhost:8080
```

## Шаг 4: Запуск сервисов

После того как вы настроили Docker и Apache, запустите контейнеры:

```bash
docker-compose up -d
```

Теперь ваш Laravel проект будет доступен по адресу `http://localhost:8080`.

## Шаг 5: Дополнительные настройки

Вы можете добавить дополнительные настройки в конфигурационный файл Apache, такие как включение mod_rewrite, настройка SSL, и т.д.

```apache
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]
</IfModule>
```

Теперь вы готовы использовать Apache для работы с вашим Laravel проектом через Docker.