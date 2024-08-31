
# Настройка Swagger для Laravel

В этом руководстве описаны шаги по настройке и интеграции Swagger с Laravel для документирования API.

## Шаг 1: Установка пакета для работы с Swagger

Для интеграции Swagger с Laravel, установите пакет `zircote/swagger-php` и `swagger-lume`:

```bash
composer require "darkaonline/swagger-lume:8.*"
```

## Шаг 2: Публикация конфигураций

После установки пакета, выполните команду для публикации конфигураций и ресурсов Swagger:

```bash
php artisan swagger-lume:publish
```

Это создаст файл конфигурации `swagger-lume.php` в директории `config` и необходимые ресурсы для Swagger.

## Шаг 3: Настройка конфигурации

Откройте файл `config/swagger-lume.php` и настройте основные параметры:

```php
'defaults' => [
    'title' => env('APP_NAME', 'Laravel API') . ' Documentation',
    'description' => 'API documentation for Laravel project',
    'version' => '1.0.0',
],

'paths' => [
    'annotations' => base_path('app'),
    'docs' => storage_path('api-docs'),
    'docs_json' => 'api-docs.json',
    'docs_yaml' => 'api-docs.yaml',
    'format_to_use_for_docs' => env('LUME_FORMAT_TO_USE_FOR_DOCS', 'json'),
],
```

## Шаг 4: Создание аннотаций в коде

Используйте аннотации Swagger в ваших контроллерах и моделях для автоматического генерации документации. Пример аннотаций:

```php
/**
 * @OA\Info(
 *     title="API Documentation",
 *     version="1.0.0"
 * )
 */

/**
 * @OA\Get(
 *     path="/api/users",
 *     @OA\Response(response="200", description="Display a listing of users")
 * )
 */
public function index()
{
    return User::all();
}
```

Эти аннотации будут использоваться для генерации документации Swagger.

## Шаг 5: Генерация документации

Для генерации документации выполните команду:

```bash
php artisan swagger-lume:generate
```

Документация будет сгенерирована и доступна по URL `/api/documentation`.

## Шаг 6: Доступ к Swagger UI

После генерации документации Swagger UI будет доступен по адресу:

```
http://localhost:8000/api/documentation
```

Теперь вы готовы использовать Swagger для документирования API вашего Laravel проекта.