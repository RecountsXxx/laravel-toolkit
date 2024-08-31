
# Настройка Memcached для Laravel

В этом руководстве описаны шаги по настройке и подключению Memcached к Laravel с использованием Docker.

## Шаг 1: Настройка `docker-compose.yml`

Добавьте сервис для Memcached в ваш `docker-compose.yml`:

```yaml
version: '3'

services:
  memcached:
    image: memcached:latest
    container_name: memcached_container
    ports:
      - "11211:11211"
    networks:
      - sne

networks:
  sne:
    driver: bridge
```

## Шаг 2: Настройка файла `.env`

Добавьте или обновите следующие параметры в вашем `.env` файле для подключения к Memcached:

```bash
CACHE_DRIVER=memcached
MEMCACHED_HOST=memcached_container
MEMCACHED_PORT=11211
```

## Шаг 3: Настройка подключения в `config/cache.php`

Откройте файл `config/cache.php` и добавьте конфигурацию для Memcached:

```php
'memcached' => [
    'driver' => 'memcached',
    'persistent_id' => env('MEMCACHED_PERSISTENT_ID'),
    'sasl' => [
        env('MEMCACHED_USERNAME'),
        env('MEMCACHED_PASSWORD'),
    ],
    'options' => [
        // Some options are suggested to better manage the memcached server.
        // 'binary_protocol' => true,
        // 'no_block' => true,
        // 'tcp_nodelay' => true,
        // 'serializer' => 'php', // default serializer
    ],
    'servers' => [
        [
            'host' => env('MEMCACHED_HOST', '127.0.0.1'),
            'port' => env('MEMCACHED_PORT', 11211),
            'weight' => 100,
        ],
    ],
],
```

## Шаг 4: Пример использования Memcached в Laravel

Вы можете использовать Memcached как драйвер кэша в Laravel для хранения различных данных, таких как результаты запросов. Вот простой пример:

```php
use Illuminate\Support\Facades\Cache;

Route::get('/cached-data', function () {
    $value = Cache::remember('key', 60, function () {
        return DB::table('users')->get();
    });

    return $value;
});
```

Этот код сохраняет результаты запроса к таблице `users` в кэш Memcached на 60 минут.

## Шаг 5: Запуск сервисов

После того как вы настроили Docker и Memcached, запустите контейнеры:

```bash
docker-compose up -d
```

Теперь вы готовы использовать Memcached в вашем Laravel проекте с помощью Docker для управления кэшем.