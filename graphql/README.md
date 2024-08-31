
# Настройка GraphQL для Laravel

В этом руководстве описаны шаги по настройке и интеграции GraphQL с Laravel.

## Шаг 1: Установка пакета GraphQL

Для интеграции GraphQL с Laravel установите пакет `rebing/graphql-laravel`:

```bash
composer require rebing/graphql-laravel
```

## Шаг 2: Публикация конфигурации

После установки пакета выполните команду для публикации конфигурационного файла:

```bash
php artisan vendor:publish --provider="Rebing\GraphQL\GraphQLServiceProvider"
```

Это создаст файл конфигурации `graphql.php` в директории `config`.

## Шаг 3: Создание схемы и запросов

### Пример схемы

Создайте файл схемы GraphQL, например, для работы с пользователями:

```php
namespace App\GraphQL\Schemas;

use Rebing\GraphQL\Support\Facades\GraphQL;
use Rebing\GraphQL\Support\Schema as BaseSchema;

class UserSchema extends BaseSchema
{
    protected $query = [
        'users' => \App\GraphQL\Queries\UsersQuery::class,
    ];

    protected $mutation = [];
}
```

### Пример запроса

Создайте запрос для получения списка пользователей:

```php
namespace App\GraphQL\Queries;

use App\Models\User;
use GraphQL\Type\Definition\Type;
use Rebing\GraphQL\Support\Facades\GraphQL;
use Rebing\GraphQL\Support\Query;

class UsersQuery extends Query
{
    protected $attributes = [
        'name' => 'users',
    ];

    public function type(): Type
    {
        return Type::listOf(GraphQL::type('User'));
    }

    public function resolve($root, $args)
    {
        return User::all();
    }
}
```

### Пример типа

Создайте тип `User` для GraphQL:

```php
namespace App\GraphQL\Types;

use App\Models\User;
use GraphQL\Type\Definition\Type;
use Rebing\GraphQL\Support\Type as GraphQLType;

class UserType extends GraphQLType
{
    protected $attributes = [
        'name' => 'User',
        'description' => 'A user',
        'model' => User::class,
    ];

    public function fields(): array
    {
        return [
            'id' => [
                'type' => Type::nonNull(Type::int()),
                'description' => 'The ID of the user',
            ],
            'name' => [
                'type' => Type::string(),
                'description' => 'The name of the user',
            ],
            'email' => [
                'type' => Type::string(),
                'description' => 'The email of the user',
            ],
        ];
    }
}
```

## Шаг 4: Настройка конфигурации

Добавьте схемы и типы в файл `config/graphql.php`:

```php
'schemas' => [
    'default' => [
        'query' => [
            'users' => \App\GraphQL\Queries\UsersQuery::class,
        ],
        'mutation' => [],
    ],
],

'types' => [
    'User' => \App\GraphQL\Types\UserType::class,
],
```

## Шаг 5: Запросы GraphQL

Теперь вы можете отправлять запросы GraphQL к вашему серверу Laravel. Например, запрос списка пользователей может выглядеть так:

```graphql
{
  users {
    id
    name
    email
  }
}
```

## Шаг 6: Тестирование запросов

Для тестирования запросов можно использовать GraphiQL, Postman или другие инструменты для работы с GraphQL API.

Теперь вы готовы использовать GraphQL в вашем Laravel проекте для создания гибкого и мощного API.