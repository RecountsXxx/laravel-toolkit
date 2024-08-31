
# Настройка MySQL для Laravel

В этом руководстве описаны шаги по настройке и подключению базы данных MySQL к Laravel с использованием Docker и phpMyAdmin для удобного управления базой данных.

## Шаг 1: Настройка `docker-compose.yml`

Добавьте сервисы для MySQL и phpMyAdmin в ваш `docker-compose.yml`:

```yaml
version: '3'
services:
  db.mysql:
    build:
      dockerfile: ./services/mysql/Dockerfile
      context: .
    container_name: db.mysql
    environment:
      - MYSQL_ROOT_PASSWORD=password
      - MYSQL_ROOT_HOST=%
      - MYSQL_DATABASE=db
      - MYSQL_USER=user
      - MYSQL_PASSWORD=password
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

## Шаг 2: Настройка Dockerfile для MySQL

Создайте Dockerfile для MySQL:

```dockerfile
# services/mysql/Dockerfile
FROM mysql:8.0
```

## Шаг 3: Настройка файла `.env`

Добавьте или обновите следующие параметры в вашем `.env` файле для подключения к MySQL:

```bash
DB_CONNECTION=mysql
DB_HOST=db.mysql
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=password
```

## Шаг 4: Настройка подключения в `config/database.php`

Откройте файл `config/database.php` и добавьте конфигурацию для MySQL:

```php
'mysql' => [
    'driver' => 'mysql',
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', '3306'),
    'database' => env('DB_DATABASE', 'forge'),
    'username' => env('DB_USERNAME', 'forge'),
    'password' => env('DB_PASSWORD', ''),
    'unix_socket' => env('DB_SOCKET', ''),
    'charset' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'prefix' => '',
    'prefix_indexes' => true,
    'strict' => true,
    'engine' => null,
],
```

## Шаг 5: Пример команды в `app/Console/Commands`

Создайте консольную команду в Laravel для работы с базой данных MySQL. Например, команда для отображения списка таблиц:

```php
namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\DB;

class ListTables extends Command
{
    protected $signature = 'db:list-tables';
    protected $description = 'List all tables in the MySQL database';

    public function handle()
    {
        $tables = DB::select('SHOW TABLES');

        foreach ($tables as $table) {
            $this->info(array_values((array)$table)[0]);
        }
    }
}
```

## Шаг 6: Запуск сервисов и команд

После того как вы настроили Docker и базу данных MySQL, запустите контейнеры:

```bash
docker-compose up -d
```

Затем выполните миграции и команды:

```bash
php artisan migrate
php artisan db:list-tables
```

Теперь вы готовы использовать MySQL в вашем Laravel проекте с помощью Docker и phpMyAdmin для удобного управления базой данных.