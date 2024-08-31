
# Настройка PHP_CodeSniffer для Laravel

В этом руководстве описаны шаги по настройке и использованию PHP_CodeSniffer для анализа кода в Laravel.

## Шаг 1: Установка PHP_CodeSniffer

Установите PHP_CodeSniffer с помощью Composer:

```bash
composer require --dev squizlabs/php_codesniffer
```

## Шаг 2: Настройка стандартов кода

По умолчанию PHP_CodeSniffer использует стандарт PSR-12. Вы можете настроить стандарт кода, который хотите использовать:

```bash
vendor/bin/phpcs --config-set default_standard PSR12
```

Вы также можете выбрать другой стандарт, например, PSR-2, или использовать свой собственный файл правил:

```bash
vendor/bin/phpcs --config-set default_standard PSR2
```

## Шаг 3: Анализ кода

Чтобы проанализировать код вашего Laravel проекта, выполните следующую команду:

```bash
vendor/bin/phpcs --standard=PSR12 app/
```

Эта команда проверит все файлы в директории `app/` на соответствие стандарту PSR-12.

## Шаг 4: Автоматическое исправление кода

Вы можете автоматически исправить некоторые ошибки кода с помощью PHP_CodeSniffer:

```bash
vendor/bin/phpcbf --standard=PSR12 app/
```

Эта команда попытается исправить любые проблемы, которые она может, в директории `app/`.

## Шаг 5: Интеграция в CI/CD

Для автоматизации анализа кода в CI/CD можно добавить команду проверки в ваши сценарии CI/CD, например, в GitHub Actions или GitLab CI:

```yaml
# Пример для GitHub Actions

jobs:
  phpcs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: 7.4
      - name: Install dependencies
        run: composer install
      - name: Run PHP_CodeSniffer
        run: vendor/bin/phpcs --standard=PSR12 app/
```

Теперь вы можете использовать PHP_CodeSniffer для поддержания качества кода в вашем Laravel проекте.