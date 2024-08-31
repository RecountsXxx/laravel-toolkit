
# Настройка RabbitMQ для Laravel и Взаимодействие между Сервисами

## Установка библиотеки php-amqplib

Для работы с RabbitMQ в Laravel и PHP используйте библиотеку `php-amqplib/php-amqplib`. Установите её с помощью Composer:

```bash
composer require php-amqplib/php-amqplib:^3.7
```

---

## Раздел 1: Использование RabbitMQ для Очередей в Laravel

Этот раздел посвящен настройке RabbitMQ для обработки очередей в Laravel и выполнению заданий (jobs).

### Шаг 1: Настройка RabbitMQ в Docker

Добавьте RabbitMQ как сервис в `docker-compose.yml`:

```yaml
version: '3'
services:
  rabbitmq:
    image: rabbitmq:management
    container_name: rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
    networks:
      - sne

networks:
  sne:
    driver: bridge
```

Запустите RabbitMQ:

```bash
docker-compose up -d rabbitmq
```

### Шаг 2: Настройка файла `.env`

Добавьте следующие параметры в файл `.env` вашего Laravel проекта:

```bash
QUEUE_CONNECTION=rabbitmq
RABBITMQ_HOST=rabbitmq
RABBITMQ_PORT=5672
RABBITMQ_LOGIN=guest
RABBITMQ_PASSWORD=guest
RABBITMQ_QUEUE=default
```

### Шаг 3: Настройка RabbitMQ в `config/queue.php`

Добавьте конфигурацию для подключения к RabbitMQ в файле `config/queue.php`:

```php
'connections' => [
    'rabbitmq' => [
        'driver' => 'rabbitmq',
        'host' => env('RABBITMQ_HOST', 'rabbitmq'),
        'port' => env('RABBITMQ_PORT', 5672),
        'vhost' => env('RABBITMQ_VHOST', '/'),
        'login' => env('RABBITMQ_LOGIN', 'guest'),
        'password' => env('RABBITMQ_PASSWORD', 'guest'),
        'queue' => env('RABBITMQ_QUEUE', 'default'),
        'options' => [
            'exchange' => [
                'name' => env('RABBITMQ_EXCHANGE_NAME', null),
                'type' => env('RABBITMQ_EXCHANGE_TYPE', 'direct'),
                'durable' => env('RABBITMQ_EXCHANGE_DURABLE', true),
            ],
            'queue' => [
                'exchange_routing_key' => env('RABBITMQ_QUEUE_ROUTING_KEY', null),
            ],
        ],
    ],
],
```

### Шаг 4: Настройка Supervisor для обработки очередей

Создайте конфигурацию Supervisor, чтобы автоматически запускать обработку очередей:

```ini
[program:laravel-rabbitmq-queue]
command=php /var/www/html/artisan queue:work --sleep=3 --tries=3
directory=/var/www/html
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/var/www/html/storage/logs/rabbitmq-queues.log
```

### Шаг 5: Запуск заданий (jobs) через RabbitMQ

Теперь, когда RabbitMQ настроен, а Supervisor обрабатывает очереди, вы можете использовать команды Laravel для запуска и обработки заданий:

Создайте задание (job):

```bash
php artisan make:job ExampleJob
```

Вызовите это задание в вашем коде:

```php
ExampleJob::dispatch()->onQueue('default');
```

Supervisor будет автоматически обрабатывать задания в очереди `default`, используя RabbitMQ.

---

## Раздел 2: Взаимодействие между Сервисами через RabbitMQ

Этот раздел посвящен настройке и использованию RabbitMQ для взаимодействия между сервисами.

### Шаг 1: Настройка сервисов в Docker Compose

В `docker-compose.yml` добавьте конфигурацию для сервисов и RabbitMQ:

```yaml
version: '3'
services:
  laravel.service.1:
    build:
      dockerfile: Dockerfile_api
      context: service-1/backend
    container_name: laravel.service.1
    volumes:
      - ./service-1/backend/src:/var/www/html
    ports:
      - '8001:8001'
    env_file:
      - ./.env
    networks:
      - sne

  laravel.service.2:
    build:
      dockerfile: Dockerfile_api
      context: service-2/backend
    container_name: laravel.service.2
    volumes:
      - ./service-2/backend/src:/var/www/html
    ports:
      - '8002:8002'
    env_file:
      - ./.env
    networks:
      - sne

  rabbitmq:
    image: rabbitmq:management
    container_name: rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
    networks:
      - sne

networks:
  sne:
    driver: bridge
```

### Шаг 2: Пример Продюсера в `service-1`

В `/service-1/backend/src/Producer.php`:

```php
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$connection = new AMQPStreamConnection('rabbitmq', 5672, 'guest', 'guest');
$channel = $connection->channel();

$channel->queue_declare('service_queue', false, false, false, false);

$msg = new AMQPMessage('Message from Service 1');
$channel->basic_publish($msg, '', 'service_queue');

echo " [x] Service 1 sent a message
";

$channel->close();
$connection->close();
```

### Шаг 3: Пример Консюмера в `service-2`

В `/service-2/backend/src/Consumer.php`:

```php
use PhpAmqpLib\Connection\AMQPStreamConnection;

$connection = new AMQPStreamConnection('rabbitmq', 5672, 'guest', 'guest');
$channel = $connection->channel();

$channel->queue_declare('service_queue', false, false, false, false);

echo " [*] Service 2 is waiting for messages
";

$callback = function ($msg) {
    echo ' [x] Service 2 received: ', $msg->body, "
";
};

$channel->basic_consume('service_queue', '', false, true, false, false, $callback);

while ($channel->is_consuming()) {
    $channel->wait();
}

$channel->close();
$connection->close();
```