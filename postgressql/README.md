
# Настройка PostgreSQL для Laravel

В этом руководстве описаны шаги по настройке и подключению базы данных PostgreSQL к Laravel с использованием Docker и pgAdmin для удобного управления базой данных.

## Шаг 1: Настройка `docker-compose.yml`

Добавьте сервисы для PostgreSQL и pgAdmin в ваш `docker-compose.yml`:

```yaml
version: '3'
services:
  postgres:
    image: postgres:14
    container_name: postgres_db
    environment:
      POSTGRES_USER: laravel
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: laravel
    volumes:
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - sne

  pgadmin:
    image: dpage/pgadmin4
    container_name: pgadmin
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: admin
    ports:
      - "5050:80"
    depends_on:
      - postgres
    networks:
      - sne

volumes:
  postgres-data:

networks:
  sne:
    driver: bridge
```

## Шаг 2: Настройка файла `.env`

Добавьте или обновите следующие параметры в вашем `.env` файле для подключения к PostgreSQL:

```bash
DB_CONNECTION=pgsql
DB_HOST=postgres_db
DB_PORT=5432
DB_DATABASE=laravel
DB_USERNAME=laravel
DB_PASSWORD=secret
```

## Шаг 3: Настройка подключения в `config/database.php`

Откройте файл `config/database.php` и добавьте конфигурацию для PostgreSQL:

```php
'pgsql' => [
    'driver' => 'pgsql',
    'url' => env('DATABASE_URL'),
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', '5432'),
    'database' => env('DB_DATABASE', 'forge'),
    'username' => env('DB_USERNAME', 'forge'),
    'password' => env('DB_PASSWORD', ''),
    'charset' => 'utf8',
    'prefix' => '',
    'prefix_indexes' => true,
    'search_path' => 'public',
    'sslmode' => 'prefer',
],
```

## Шаг 4: Пример команды в `app/Console/Commands`

Создайте консольную команду в Laravel для работы с базой данных PostgreSQL. Например, команда для отображения списка таблиц:

```php
namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\DB;

class ListTables extends Command
{
    protected $signature = 'db:list-tables';
    protected $description = 'List all tables in the PostgreSQL database';

    public function handle()
    {
        $tables = DB::select('SELECT table_name FROM information_schema.tables WHERE table_schema = 'public'');

        foreach ($tables as $table) {
            $this->info($table->table_name);
        }
    }
}
```

## Шаг 5: Запуск сервисов и команд

После того как вы настроили Docker и базу данных PostgreSQL, запустите контейнеры:

```bash
docker-compose up -d
```

Затем выполните миграции и команды:

```bash
php artisan migrate
php artisan db:list-tables
```

Теперь вы готовы использовать PostgreSQL в вашем Laravel проекте с помощью Docker и pgAdmin для удобного управления базой данных.