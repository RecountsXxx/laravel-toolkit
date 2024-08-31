
# Настройка Apache Kafka для Laravel

В этом руководстве описаны шаги по настройке и интеграции Apache Kafka с Laravel.

## Шаг 1: Настройка Kafka с помощью Docker

Добавьте сервисы для Apache Kafka и Zookeeper в ваш `docker-compose.yml`:

```yaml
version: '3'

services:
  zookeeper:
    image: wurstmeister/zookeeper:3.4.6
    container_name: zookeeper
    ports:
      - "2181:2181"
    networks:
      - sne

  kafka:
    image: wurstmeister/kafka:2.13-2.7.0
    container_name: kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092
    networks:
      - sne

networks:
  sne:
    driver: bridge
```

## Шаг 2: Установка пакета для работы с Kafka

Для интеграции Kafka с Laravel, установите пакет `nmoksha/kafka-laravel`:

```bash
composer require nmoksha/kafka-laravel
```

## Шаг 3: Настройка конфигурации Kafka

После установки пакета, опубликуйте конфигурационный файл:

```bash
php artisan vendor:publish --provider="Nmoksha\Kafka\KafkaServiceProvider"
```

Это создаст файл конфигурации `kafka.php` в директории `config`.

## Шаг 4: Пример использования Kafka

### Пример продюсера

Создайте продюсер для отправки сообщений в Kafka:

```php
namespace App\Console\Commands;

use Illuminate\Console\Command;
use Nmoksha\Kafka\Facades\Kafka;

class ProduceKafkaMessage extends Command
{
    protected $signature = 'kafka:produce';
    protected $description = 'Produce a message to Kafka topic';

    public function handle()
    {
        $producer = Kafka::publishOn('test-topic')
            ->withMessage(['key' => 'value'])
            ->send();

        $this->info('Message sent to Kafka topic');
    }
}
```

### Пример консюмера

Создайте консюмера для обработки сообщений из Kafka:

```php
namespace App\Console\Commands;

use Illuminate\Console\Command;
use Nmoksha\Kafka\Facades\Kafka;

class ConsumeKafkaMessage extends Command
{
    protected $signature = 'kafka:consume';
    protected $description = 'Consume messages from Kafka topic';

    public function handle()
    {
        $consumer = Kafka::consume('test-topic')
            ->withHandler(function ($message) {
                // Обработка сообщения
                $this->info('Received message: ' . json_encode($message->payload));
            })
            ->start();
    }
}
```

## Шаг 5: Запуск сервисов и команд

После того как вы настроили Docker и Kafka, запустите контейнеры:

```bash
docker-compose up -d
```

Затем запустите продюсера и консюмера:

```bash
php artisan kafka:produce
php artisan kafka:consume
```

Теперь вы готовы использовать Apache Kafka в вашем Laravel проекте для обработки сообщений и взаимодействия с микросервисами.