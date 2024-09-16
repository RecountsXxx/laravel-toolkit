# Laravel Solid Example

## Business Features

- Implement CRUD functionality for managing company departments
- Implement CRUD functionality for managing employees assigned to departments
- Implement report generation for departments, such as:
    - List all employees in each department
    - List all employees with their salaries
    - List all employees with their roles and responsibilities within the department.

## Run project

Install dependencies:

    composer install

Copy .env file:

    cp .env.example .env

Generate app key:

    php artisan key:generate

Run migrations:

    php artisan migrate

Run tests (optional):

    ./vendor/bin/phpunit
