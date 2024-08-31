
# Настройка Socket.IO для Laravel

В этом руководстве описаны шаги по настройке и использованию Socket.IO с Laravel с использованием Docker.

## Шаг 1: Установка необходимых пакетов

Установите Node.js и необходимые пакеты:

```bash
npm install socket.io
npm install laravel-echo
npm install --save-dev laravel-echo-server
```

## Шаг 2: Настройка `docker-compose.yml`

Добавьте сервис для Node.js в ваш `docker-compose.yml`, если он еще не добавлен:

```yaml
version: '3'

services:
  nodejs:
    image: node:latest
    container_name: nodejs_container
    volumes:
      - .:/app
    working_dir: /app
    command: bash -c "npm install && npm run watch"
    networks:
      - sne

networks:
  sne:
    driver: bridge
```

## Шаг 3: Настройка Laravel Echo Server

Создайте конфигурационный файл для Laravel Echo Server:

```bash
npx laravel-echo-server init
```

В процессе инициализации вам будет предложено настроить сервер. Убедитесь, что вы указали правильные параметры для работы с Redis или другим брокером сообщений, который вы планируете использовать.

Пример файла `laravel-echo-server.json`:

```json
{
    "authHost": "http://localhost",
    "authEndpoint": "/broadcasting/auth",
    "clients": [
        {
            "appId": "yourAppId",
            "key": "yourAppKey"
        }
    ],
    "database": "redis",
    "databaseConfig": {
        "redis": {
            "host": "redis",
            "port": "6379"
        }
    },
    "host": null,
    "port": "6001",
    "protocol": "http",
    "socketio": {},
    "secureOptions": 67108864,
    "sslCertPath": "",
    "sslKeyPath": "",
    "sslCertChainPath": "",
    "sslPassphrase": "",
    "subscribers": {
        "http": true,
        "redis": true
    },
    "apiOriginAllow": {
        "allowCors": true,
        "allowOrigin": "*",
        "allowMethods": "GET, POST",
        "allowHeaders": "Origin, Content-Type, X-Requested-With, Accept"
    }
}
```

## Шаг 4: Настройка Broadcast в Laravel

Настройте драйвер вещания в файле `.env`:

```bash
BROADCAST_DRIVER=redis
QUEUE_CONNECTION=redis
REDIS_HOST=redis
```

Убедитесь, что в `config/broadcasting.php` используется правильный драйвер:

```php
'connections' => [
    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
    ],
],
```

## Шаг 5: Настройка Socket.IO клиента

Обновите файл `resources/js/bootstrap.js`, чтобы включить Laravel Echo и Socket.IO:

```javascript
import Echo from 'laravel-echo';

window.io = require('socket.io-client');

window.Echo = new Echo({
    broadcaster: 'socket.io',
    host: window.location.hostname + ':6001'
});
```

## Шаг 6: Запуск сервисов

После того как вы настроили Docker и Node.js, запустите контейнеры:

```bash
docker-compose up -d
```

Также запустите Laravel Echo Server:

```bash
npx laravel-echo-server start
```

Теперь ваш Laravel проект настроен для использования Socket.IO с помощью Laravel Echo и Laravel Echo Server.