
# Настройка MongoDB для Laravel

В этом руководстве описаны шаги по настройке и подключению базы данных MongoDB к Laravel с использованием Docker и mongo-express для удобного управления базой данных.

## Шаг 1: Настройка `docker-compose.yml`

Добавьте сервисы для MongoDB и mongo-express в ваш `docker-compose.yml`:

```yaml
version: '3'

services:
  mongodb:
    image: mongo:latest
    container_name: mongodb_container
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: adminpassword
      MONGO_INITDB_DATABASE: mydatabase
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
      - mongo_config:/data/configdb
    networks:
      - sne

  mongo-express:
    image: mongo-express:latest
    container_name: mongo_express
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: admin
      ME_CONFIG_MONGODB_ADMINPASSWORD: adminpassword
      ME_CONFIG_MONGODB_SERVER: mongodb_container
    ports:
      - "8081:8081"
    depends_on:
      - mongodb
    networks:
      - sne

volumes:
  mongo_data:
  mongo_config:

networks:
  sne:
    driver: bridge
```

## Шаг 2: Настройка файла `.env`

Добавьте или обновите следующие параметры в вашем `.env` файле для подключения к MongoDB:

```bash
DB_CONNECTION=mongodb
DB_HOST=mongodb_container
DB_PORT=27017
DB_DATABASE=mydatabase
DB_USERNAME=admin
DB_PASSWORD=adminpassword
```

## Шаг 3: Настройка подключения в `config/database.php`

Для подключения MongoDB в Laravel, необходимо установить пакет `jenssegers/laravel-mongodb`:

```bash
composer require jenssegers/mongodb
```

Затем откройте файл `config/database.php` и добавьте конфигурацию для MongoDB:

```php
'mongodb' => [
    'driver' => 'mongodb',
    'host' => env('DB_HOST', 'localhost'),
    'port' => env('DB_PORT', 27017),
    'database' => env('DB_DATABASE'),
    'username' => env('DB_USERNAME'),
    'password' => env('DB_PASSWORD'),
    'options' => [
        'database' => env('DB_AUTHENTICATION_DATABASE', 'admin'), // Optional: Define the authentication database
    ],
],
```

## Шаг 4: Пример модели и команды в Laravel

### Пример модели

Создайте простую модель для работы с коллекцией MongoDB. Например, модель для пользователей:

```php
namespace App\Models;

use Jenssegers\Mongodb\Eloquent\Model;

class User extends Model
{
    protected $connection = 'mongodb';
    protected $collection = 'users';

    protected $fillable = [
        'name', 'email', 'password',
    ];
}
```

### Пример консольной команды

Создайте консольную команду в Laravel для работы с базой данных MongoDB. Например, команда для отображения всех пользователей:

```php
namespace App\Console\Commands;

use Illuminate\Console\Command;
use App\Models\User;

class ListUsers extends Command
{
    protected $signature = 'db:list-users';
    protected $description = 'List all users in the MongoDB database';

    public function handle()
    {
        $users = User::all();

        foreach ($users as $user) {
            $this->info($user->name . ' (' . $user->email . ')');
        }
    }
}
```

## Шаг 5: Запуск сервисов и команд

После того как вы настроили Docker и базу данных MongoDB, запустите контейнеры:

```bash
docker-compose up -d
```

Затем выполните команды:

```bash
php artisan db:list-users
```

Теперь вы готовы использовать MongoDB в вашем Laravel проекте с помощью Docker и mongo-express для удобного управления базой данных.