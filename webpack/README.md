
# Настройка Webpack для Laravel

В этом руководстве описаны шаги по настройке и интеграции Webpack с Laravel для управления фронтенд-активами.

## Шаг 1: Установка Webpack и зависимостей

Для начала установите Webpack и необходимые зависимости с помощью npm или yarn:

```bash
npm install --save-dev webpack webpack-cli
npm install --save-dev css-loader style-loader
npm install --save-dev babel-loader @babel/core @babel/preset-env
npm install --save-dev file-loader
```

Или используя yarn:

```bash
yarn add --dev webpack webpack-cli
yarn add --dev css-loader style-loader
yarn add --dev babel-loader @babel/core @babel/preset-env
yarn add --dev file-loader
```

## Шаг 2: Настройка конфигурации Webpack

Создайте файл `webpack.config.js` в корневой директории вашего Laravel проекта:

```js
const path = require('path');

module.exports = {
    entry: './resources/js/app.js',
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'public/js'),
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: ['style-loader', 'css-loader'],
            },
            {
                test: /\.js$/,
                exclude: /node_modules/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['@babel/preset-env'],
                    },
                },
            },
            {
                test: /\.(png|svg|jpg|gif)$/,
                use: ['file-loader'],
            },
        ],
    },
};
```

## Шаг 3: Настройка Babel

Создайте файл конфигурации Babel `.babelrc`:

```json
{
    "presets": ["@babel/preset-env"]
}
```

## Шаг 4: Настройка скриптов в package.json

Добавьте скрипты для запуска Webpack в ваш `package.json`:

```json
"scripts": {
    "dev": "webpack --mode development",
    "prod": "webpack --mode production"
}
```

## Шаг 5: Использование Webpack в Laravel

Создайте файл `app.js` в директории `resources/js`:

```js
import '../css/app.css';

console.log('Webpack is working!');
```

Создайте файл `app.css` в директории `resources/css`:

```css
body {
    background-color: #f0f0f0;
}
```

Теперь вы можете запустить Webpack для сборки ваших файлов:

```bash
npm run dev
```

Или для продакшн сборки:

```bash
npm run prod
```

## Шаг 6: Подключение в шаблоне Laravel

Подключите сгенерированный `bundle.js` в вашем Blade шаблоне:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Laravel with Webpack</title>
    <link rel="stylesheet" href="{{ asset('css/app.css') }}">
</head>
<body>
    <script src="{{ asset('js/bundle.js') }}"></script>
</body>
</html>
```

Теперь ваш Laravel проект настроен для работы с Webpack для управления и сборки фронтенд-активов.