
# Laravel Project Setup with Docker, Nginx, MySQL, and Redis

Этот документ описывает процесс настройки и запуска проекта Laravel с использованием Docker, Nginx, MySQL, и Redis. Включены примеры конфигураций для всех сервисов и инструкции по их запуску.

## 1. Структура проекта

Проект организован следующим образом:

- **backend/**: Исходный код Laravel и конфигурации PHP.
- **services/**: Конфигурации Docker для MySQL, Nginx и Redis.
- **volumes/**: Данные для MySQL и Redis, а также файлы `.env`.

## 2. Nginx конфигурация

Файл `nginx.conf` настроен для проксирования запросов на Laravel API:

```nginx
worker_processes 4;

events {
  worker_connections 1024;
}

http {
    client_max_body_size 20M;
    client_body_buffer_size 128k;

    server {
        listen 80;

        location / {
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
              proxy_set_header Host $host;
              proxy_pass http://laravel.api:8000;
        }
    }
}
```

## 3. Docker Compose

Конфигурация `docker-compose.yml` для управления несколькими сервисами:

```yaml
version: '3'
services:

  laravel.api:
    build:
      dockerfile: Dockerfile_api
      context: backend/
    container_name: laravel.api
    volumes:
      - ./backend/src:/var/www/html
    ports:
      - '8000:8000'
    networks:
      - sne
    depends_on:
      - db.mysql

  nginx.router:
    build:
      dockerfile: ./services/nginx/Dockerfile
      context: .
    volumes:
      - ./services/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    ports:
      - '80:80'
    networks:
      - sne
    depends_on:
      - laravel.api
      - db.mysql

  db.redis:
    build:
      dockerfile: ./services/redis/Dockerfile
      context: .
    container_name: db.redis
    ports:
      - "6379:6379"
    volumes:
      - ./volumes/db/redis/jobs:/data
    networks:
      - sne

  db.mysql:
    build:
      dockerfile: ./services/mysql/Dockerfile
      context: .
    container_name: db.mysql
    env_file:
      - .env
    volumes:
      - ./volumes/db/mysql/main:/var/lib/mysql
    ports:
      - '3306:3306'
    networks:
      - sne

  phpmyadmin:
    image: phpmyadmin
    ports:
      - 9090:80
    environment:
      - PMA_HOST=db.mysql
      - PMA_PORT=3306
    networks:
      - sne

networks:
  sne:
    driver: bridge
```

## 4. Подключение Laravel к Redis и MySQL

Пример настройки подключения к Redis и MySQL в файле `.env`:

```env
DB_CONNECTION=mysql
DB_HOST=db.mysql
DB_PORT=3306
DB_DATABASE=your_database_name
DB_USERNAME=your_username
DB_PASSWORD=your_password

CACHE_DRIVER=redis
REDIS_HOST=db.redis
REDIS_PASSWORD=null
REDIS_PORT=6379
```

### Настройка Redis в `config/queue.php`

```php
'redis' => [
    'driver' => 'redis',
    'connection' => 'default',
    'queue' => env('REDIS_QUEUE', 'default'),
    'retry_after' => 90,
    'block_for' => null,
],
```

## 5. Запуск проекта

Для запуска всех сервисов выполните команду:

```bash
docker-compose up -d
```

После запуска контейнеров проект будет доступен по адресу `http://localhost`, а phpMyAdmin по адресу `http://localhost:9090`.

Теперь ваш Laravel проект настроен для работы с Docker, Nginx, MySQL и Redis.
