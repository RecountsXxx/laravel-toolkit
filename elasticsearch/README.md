
# Настройка Elasticsearch для Laravel

В этом руководстве описаны шаги по настройке и подключению Elasticsearch к Laravel с использованием Docker.

## Шаг 1: Настройка `docker-compose.yml`

Добавьте сервисы для Elasticsearch и Kibana в ваш `docker-compose.yml`:

```yaml
version: '3'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.9.3
    container_name: elasticsearch_container
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
      - "9300:9300"
    networks:
      - sne

  kibana:
    image: docker.elastic.co/kibana/kibana:7.9.3
    container_name: kibana_container
    ports:
      - "5601:5601"
    environment:
      ELASTICSEARCH_HOSTS: "http://elasticsearch:9200"
    depends_on:
      - elasticsearch
    networks:
      - sne

networks:
  sne:
    driver: bridge
```

## Шаг 2: Настройка файла `.env`

Добавьте или обновите следующие параметры в вашем `.env` файле для подключения к Elasticsearch:

```bash
ELASTICSEARCH_HOST=http://elasticsearch:9200
```

## Шаг 3: Установка пакета Elasticsearch для Laravel

Установите пакет `elasticsearch/elasticsearch` с помощью Composer:

```bash
composer require elasticsearch/elasticsearch
```

## Шаг 4: Настройка подключения в `config/services.php`

Добавьте конфигурацию для подключения к Elasticsearch в `config/services.php`:

```php
'elastic' => [
    'hosts' => [
        env('ELASTICSEARCH_HOST', 'http://localhost:9200'),
    ],
],
```

## Шаг 5: Пример использования Elasticsearch в Laravel

Создайте простой пример использования Elasticsearch для поиска данных. Например, создадим сервис для поиска пользователей:

```php
namespace App\Services;

use Elasticsearch\ClientBuilder;

class UserSearchService
{
    protected $client;

    public function __construct()
    {
        $this->client = ClientBuilder::create()
            ->setHosts(config('services.elastic.hosts'))
            ->build();
    }

    public function search($query)
    {
        $params = [
            'index' => 'users',
            'body' => [
                'query' => [
                    'match' => [
                        'name' => $query,
                    ],
                ],
            ],
        ];

        return $this->client->search($params);
    }
}
```

Вызовите этот сервис в контроллере:

```php
namespace App\Http\Controllers;

use App\Services\UserSearchService;
use Illuminate\Http\Request;

class UserController extends Controller
{
    protected $searchService;

    public function __construct(UserSearchService $searchService)
    {
        $this->searchService = $searchService;
    }

    public function search(Request $request)
    {
        $results = $this->searchService->search($request->input('query'));

        return response()->json($results);
    }
}
```

## Шаг 6: Запуск сервисов

После того как вы настроили Docker и Elasticsearch, запустите контейнеры:

```bash
docker-compose up -d
```

Теперь вы готовы использовать Elasticsearch в вашем Laravel проекте с помощью Docker для поиска и анализа данных.