
# Настройка PHPUnit для Laravel

В этом руководстве описаны шаги по настройке и использованию PHPUnit для тестирования в Laravel.

## Шаг 1: Установка PHPUnit

Laravel поставляется с PHPUnit по умолчанию. Чтобы убедиться, что он установлен, выполните следующую команду:

```bash
composer install
```

Если PHPUnit не установлен, его можно установить отдельно:

```bash
composer require --dev phpunit/phpunit
```

## Шаг 2: Настройка тестовой базы данных

Добавьте или обновите следующие параметры в вашем `.env` файле для тестирования:

```bash
DB_CONNECTION=sqlite
DB_DATABASE=:memory:
```

Для других баз данных (например, MySQL), настройте параметры для тестовой среды:

```bash
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=testing
DB_USERNAME=root
DB_PASSWORD=
```

## Шаг 3: Настройка phpunit.xml

Файл `phpunit.xml` уже настроен в Laravel. Убедитесь, что указаны правильные настройки для вашей тестовой среды, особенно для базы данных:

```xml
<env name="DB_CONNECTION" value="sqlite"/>
<env name="DB_DATABASE" value=":memory:"/>
```

Если вы используете другую базу данных:

```xml
<env name="DB_CONNECTION" value="mysql"/>
<env name="DB_DATABASE" value="testing"/>
<env name="DB_USERNAME" value="root"/>
<env name="DB_PASSWORD" value=""/>
```

## Шаг 4: Написание тестов

Laravel поддерживает написание тестов как для функционального, так и для модульного тестирования. Например, создадим простой тест для проверки главной страницы:

```php
namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic test example.
     *
     * @return void
     */
    public function test_home_page_is_accessible()
    {
        $response = $this->get('/');

        $response->assertStatus(200);
    }
}
```

Этот тест проверяет, что главная страница доступна и возвращает статус 200.

## Шаг 5: Запуск тестов

Запустите все тесты в вашем проекте с помощью следующей команды:

```bash
php artisan test
```

Или напрямую через PHPUnit:

```bash
vendor/bin/phpunit
```

Если вы хотите запустить конкретный тестовый файл:

```bash
vendor/bin/phpunit --filter ExampleTest
```

Теперь вы готовы к использованию PHPUnit для тестирования вашего Laravel проекта.