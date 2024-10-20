# Рабочая среда с Laravel, Vite and Vue

В этом гайде я расскажу о том, как можно настроить проект на Laravel добавив к нему Vue. В релизе [v9.19](https://github.com/laravel/framework/releases/tag/v9.19.0) заменили [Laravel Mix](https://laravel-mix.com/) на [Vite](https://vitejs.dev). По некоторой информации от моих коллег Vite еще сыроват, однако скорость у него сумасшедшая. Ну а если речь идет не о супер серьезном проекте, то почему-бы и нет?

Важно понимать, что мой гайд не будет основан на готовых решениях типа - [Laravel Breeze](https://laravel.com/docs/9.x/starter-kits#breeze-and-inertia). Наша задача развернуть все самостоятельно. 

Я использовал Laravel версии `9.45.1`

## Установка Vue

Vite идет из коробки вместе с Laravel, а значит нам достаточно установить и настроить Vue. Я буду использовать Vue 3. Некоторым это может не понравится, но я могу обосновать свое решение. Сам по себе Vue 3 колоссально не отличается от второй версии. Он позволяет использовать как [Composition API](https://vuejs.org/guide/extras/composition-api-faq.html), так и старый добрый [Options API](https://v3.ru.vuejs.org/ru/api/options-api.html). Однако, если третья версия все же не устраивает по каким-то причинам, например важный для вас набор пакетов не поддерживает Vue 3, то можно использовать вторую.

Установим Vue и Vue Loader.

```shell
npm i vue@next vue-loader@next
```

Также не забудем про драйвер, который позволит использовать Vue вместе с Vite.

```shell
npm i @vitejs/plugin-vue
```

## Настройка Vite

Теперь необходимо настроить сам Vite. В корне проекта начиная с версии 9.19 появился файл - `vite.config.js`

Вот его стандартное содержимое:

```javascript
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
});
```

Добавим плагин для Vue, который мы установили чуть раньше:

```diff
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
+import vue from '@vitejs/plugin-vue'

export default defineConfig({
    plugins: [
+        vue(),
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
});
```

## Инициализация Vue

Теперь давайте сделаем все необходимое, чтобы запустить наше приложение. 
Для начала добавим первый .vue компонент, который будет выступать основным шаблоном для нашего приложения. Я добавлю его сюда - `resources/js/vue/App.vue`.

Вот его код, чтобы вам было не скучно:

```vue
<template>
  <h1>Hello, world!</h1>
</template>

<script>
export default {
  name: "App"
}
</script>
```

Теперь откроем `resources/js/app.js` и добавим немного кода.

```javascript
import './bootstrap';

// импортируем экземпляр Vue приложения
import { createApp } from 'vue';

// наш тот самый первый компонент
import App from './vue/App.vue';

// и немного дефолтных мелочей
createApp(App)
    .mount('#app')
```

Последнее - это добавление нашего скрипта на саму страничку. С появлением Vite появилась и директива `@vite()`, которая нам сейчас пригодится.

Возьмем `welcome.blade.php` и напишем в нем вот такой код:

```html
<!doctype html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
<!-- элемент, в котором будет Vue приложение -->
<div id="app"></div>
<!-- импорт всего необходимого через Vite -->
@vite('resources/js/app.js')
</body>
</html>
```

## Запуск приложения

Я покажу на примере `php artisan serve`, потому-что это очень распространенный вариант, однако это также хорошо работает и c [Laravel Sail](https://laravel.com/docs/9.x/sail), который я использую постоянно.

Выполняем команду:

```shell
npm run dev
```

На экране появится нечто подобное:

```text
  VITE v4.0.3  ready in 212 ms

  ➜  Local:   http://localhost:5174/
  ➜  Network: use --host to expose
  ➜  press h to show help

  LARAVEL v9.45.1  plugin v0.7.3

  ➜  APP_URL: http://localhost
```

Наверняка вас интересует что это такое - `http://localhost:5174`. Как вы могли догадаться это далеко не наше приложение, а _...the Vite development server that provides Hot Module Replacement for your Laravel application..._ Именно эта штука обеспечивает супер быструю горячую перезагрузку после редактирования кода, а также через этот хост подтягиваются все скрипты и стили. 

Непосредственно нашего приложения это - `APP_URL`. Его нужно подкорректировать, указав тот хост, который мы получим после запуска с помощью `php artisan serve`. Как правило, это - `http://127.0.0.1:8000`.

Когда мы настроили конфиг, запустили `npm run dev` и `php artisan serve` - наше приложение готово к работе. Попробуйте изменить что-нибудь в `App.vue` и проверить горячую перезагрузку.

Если у вас ничего не работает (как это было у меня), то попробуйте немного дополнить ваш конфиг Vite.

```diff
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue'

export default defineConfig({
+    server: {
+        hmr: {
+            host: 'localhost',
+        },
+    },
    plugins: [
        vue(),
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
});
```
## Vue Router
Для начала установим сам Vue Router.

```shell
npm i vue-router@4
```
Теперь в папку `src` добавим все необходимое, чтобы наш маршрутизатор заработал. Я создал папку `router` и поместил в нее несколько файликов:

- `index.js` - это будет сам роутер, где мы задаем конфигурацию и т.п.
- `routes.js` - тут мы будем хранить сами маршруты

Я также сразу добавлю несколько страниц в папку `pages`, которые по своей сути обычные vue-компоненты.

Пример:

```vue
<template>
  Home page
</template>

<script>
export default {
  name: 'Home'
}
</script>
```
Итого получаем вот такую структуру файлов:

```
├── App.vue
├── pages
│   ├── about.vue
│   └── home.vue
└── router
    ├── index.js
    └── routes.js
```
Для начала опишем наши маршруты в файле `router/routes.js`

```js
import Home from '../pages/home.vue'
import About from '../pages/about.vue';

export default [
    {
        path: '/',
        name: 'Home',
        component: Home,
    },
    {
        path: '/about',
        name: 'About',
        component: About,
    },
]
```

Теперь вернемся к `router/index.js` и закончим настройку нашего роутера.

```js
import {createWebHistory, createRouter} from 'vue-router';
import routes from './routes'

const router = createRouter({
    history: createWebHistory(),
    routes,
});

export default router;
```

Я не буду пояснять за каждую строчку, так как это обычная инициализация vue-router, как и в других приложениях.

Осталось самое главное - заюзать наш роутер в самом Vue. Для этого откроем `resources/js/app.js` и немного подредактируем:

```diff
import { createApp } from 'vue';
import App from './src/App.vue';
+import router from './src/router';

createApp(App)
+   .use(router)
    .mount('#app')
```

`App.vue` также нужно поправить, потому-что ему не хватает `<router-view>` + я сразу добавлю ссылки с помощью `<router-link>`, чтобы проверить работоспособность.

```vue
<template>
  <ul>
    <li><router-link to="/">Home</router-link></li>
    <li><router-link to="/about">About</router-link></li>
  </ul>
  <router-view />
</template>

<script>
export default {
  name: 'App'
}
</script>
```

Вау! Теперь у нас есть роутер. Но что будет, если мы перейдем на страничку `/about` и обновим ее? Вероятно вы получите 404 ответ. Чтобы такого не происходило, нам нужно все запросы проксировать на наш `welcome.blade.php`, то есть на тот файл, в котором запускается наше Vue приложение. Откроем `routes/web.php` и слегка поправим наш маршрут.

```php
<?php

Route::get('{any?}', function () {
    return view('welcome');
})->where('any', '.*');
```

Вот теперь другое дело!
## Vuex
_Coming soon.._
## TypeScript
_Coming soon.._
