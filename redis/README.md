
# Пошаговая инструкция по настройке Redis для Laravel

Этот проект использует Redis для обработки очередей. Ниже представлены понятные шаги по настройке Redis и интеграции его с Laravel.

## Шаг 1: Настройка файла `.env`

Добавьте данные для подключения к Redis в файл `.env`:

```bash
REDIS_HOST=db.redis
REDIS_PASSWORD=null
REDIS_PORT=6379
QUEUE_CONNECTION=redis
```

## Шаг 2: Конфигурация Redis в файле `config/database.php`

Откройте файл `config/database.php` и добавьте следующую конфигурацию для Redis:

```php
'redis' => [
    'client' => env('REDIS_CLIENT', 'phpredis'),

    'default' => [
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => env('REDIS_DB', 0),
    ],

    'cache' => [
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => env('REDIS_CACHE_DB', 1),
    ],
],
```

## Шаг 3: Настройка очереди в файле `config/queue.php`

Добавьте новую очередь, которая будет использовать Redis:

```php
'connections' => [

    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => env('REDIS_QUEUE', 'default'),
        'retry_after' => 90,
        'block_for' => null,
    ],

],

'default' => env('QUEUE_CONNECTION', 'redis'),
```

## Шаг 4: Настройка Supervisor

Для автоматического запуска очереди через Supervisor создайте файл конфигурации:

```bash
[program:{queue_name}]
command=php /var/www/html/artisan queue:work --queue={queue_name} --sleep=1 --tries=1
directory=/var/www/html
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/var/www/html/storage/logs/{queue_name}.log
```

Замените `{queue_name}`, `{your_username}`, и путь к проекту на актуальные данные.

## Шаг 5: Использование очереди в Laravel

После настройки Redis и Supervisor вы можете использовать очереди в Laravel. Все задания будут обрабатываться через Redis.

## Шаг 6: Добавление Redis в Docker Compose

Для запуска Redis добавьте следующий сервис в ваш файл `docker-compose.yml`:

```yaml
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
```

## Пример использования

1. Создайте задание (Job) командой:  
   ```php artisan make:job NAME_JOB```

2. Запустите задание в указанной очереди:  
   ```NAME_JOB::dispatch()->onQueue(NAME_QUEUE)```

Когда вы укажете имя очереди, задание начнет выполняться в ней.

## Пример структуры проекта

Ниже представлена примерная структура проекта, включающая настройку Redis и Docker Compose:

```
redis/
│
├── backend/
│   └── src/
│   ├── Dockerfile_api
│   └── supervisor_api.conf
│
├── services/
│   ├── nginx/
│   └── redis/
│
├── volumes/
│
├── .gitignore
├── docker-compose.yml
└── README.md
```

Эта структура помогает легко управлять проектом, включая настройки Redis, Docker и конфигурационные файлы Supervisor.