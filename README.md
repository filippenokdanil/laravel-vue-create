# Vue3 + Laravel 11 + Vite.
Данный репозиторий является форком другого, который уже давно не менялся и не редактировался. Причина создания этого репозитория заключается в распространении информации о работе со Vue 3 и Laravel 11, чтобы повысить популярность данного стека технологий для разработки. Здесь я буду дополнять информацию как по связке фреймворков, так и про каждый по отдельности. 
## Оглавление
1. **Создание проекта на базе Vue 3 (frontend) и Laravel (backend)**.

   1.1. Установка Laravel.

   1.2. Настройка Laravel.

   1.3. Если вдруг laravel не видит новые методы.

   1.4. Установка API в Laravel.

   1.5. Установка Vue.

   1.6. Настройка Vite.

   1.7. Инициализация Vue.

   1.8. Запуск приложения.

   1.9. Vue Router.

   1.10. Настройка .htaccess на сервере для перенаправления на laravel.

   1.11. Как убрать build в url странице.
   
3. **Vue 3**.

	2.1. Отмотка страницы при переключении страниц.

	2.2. Изменение заголовка html страницы.

   	2.3. Установка axios, а также отлов ошибок в проекте.

4. **Laravel 11**.

	3.1. Очистка и запуск конкретного файла миграции.

Если у вас есть возможность распространить ссылку на данный репозиторий или добавить звёздочку для продвижения, я буду только рад! 
 
# 1. Создание проекта на базе Vue 3 (frontend) и Laravel (backend).
## 1.1. Установка Laravel.

Для начала необходимо выбрать дерикторию, где будет находится наш проект с Laravel. Для этого достаточно в консоли просто перейти туда при помощи команды cd:

```shell
cd ./derictory
```

Дальше нужно устоновить сам Laravel, для этого нужно при помощи композера запустить процесс, которые осуществляется установкой напрямую через composer, либо при помощи глобальной инициализации:

```shell
<!-- Напрямую -->
composer create-project laravel/laravel [projectName]

<!-- Глобально -->
composer global require laravel/installer
laravel new [projectName]
```

## 1.2. Настройка Laravel.

Дальше необходимо настроить файл .env внутри папки проекта:
```env
DB_CONNECTION=[pgsql/sqlite/mysql/mariadb]
DB_HOST=[название хоста (127.0.0.1 или PostgreSQL-16)]
DB_PORT=[порт, где размещена бд]
DB_DATABASE=[название бд]
DB_USERNAME=[имя пользователя в бд]
DB_PASSWORD=[пароль от пользователя]
```

Также нужно поменять настройки подключения в файле /config/database.php :

```php
// Здесь мы выбираем способ подключения (pgsql/sqlite/mysql/mariadb)
'default' => env('DB_CONNECTION', 'pgsql'),

// Дальше настраиваем наш модуль, который мы выбрали
'connections' => [
    'pgsql' => [
        'driver' => 'pgsql',
        'url' => env('DB_URL'),
        'host' => env('DB_HOST', 'PostgreSQL-16'),
        'port' => env('DB_PORT', '5432'),
        'database' => env('DB_DATABASE', 'mydb'),
        'username' => env('DB_USERNAME', 'postgres'),
        'password' => env('DB_PASSWORD', '123456'),
        'charset' => env('DB_CHARSET', 'utf8'),
        'prefix' => '',
        'prefix_indexes' => true,
        'search_path' => 'public',
        'sslmode' => 'prefer',
    ],
],
```

После настройки теперь необходимо создать бд с таким же названием в вашей СУБД, а после выполнить команду для развёртывания таблиц laravel:
```shell
php artisan migrate
```
Если возникнут проблемы, когда вы уже создавали подобную бд или повторно подключаетесь из другого проекта используйте очищение кеша laravel:
```shell
php artisan config:clear
```

## 1.3. Если вдруг laravel не видит новые методы.

Для этого необходимо выполнить следующие команды:

```shell
php artisan cache:clear 
php artisan config:clear
hp artisan config:cache
php artisan route:clear
php artisan view:clear
php artisan route:cache
php artisan view:cache
php artisan optimize
```

Или же можно обновить всё одной командой:

```shell
php artisan optimize:clear
```

Если ничего не помогло, попробуйте обновить автозагрузку классов composer:

```shell
composer dump-autoload
```

## 1.4. Установка API в Laravel

Для того, чтобы добавить в ваш проект laravel возможность обращение, как к API нужно ввести следующую комманду [комманду](https://laravel.com/docs/11.x/sanctum#installation):

```shell
php artisan install:api
```

Если вам после этого нужно настроить настройки cors, нужно ввести следующую команду для установки файла [cors.php](https://laravel.com/docs/11.x/sanctum#cors-and-cookies):

```shell
php artisan config:publish cors
```

Далее следует в файле /config/cors.php поменять 'allowed_origins':

```php
'allowed_origins' => ['https://localhost']
```


## 1.5. Установка Vue.

Vite идет из коробки вместе с Laravel, а значит нам достаточно установить и настроить Vue. Я буду использовать Vue 3. Некоторым это может не понравится, но я могу обосновать свое решение. Сам по себе Vue 3 колоссально не отличается от второй версии. Он позволяет использовать как [Composition API](https://vuejs.org/guide/extras/composition-api-faq.html), так и старый добрый [Options API](https://v3.ru.vuejs.org/ru/api/options-api.html). Однако, если третья версия все же не устраивает по каким-то причинам, например важный для вас набор пакетов не поддерживает Vue 3, то можно использовать вторую.

Установим Vue.

```shell
npm install vue@latest
```

Также не забудем про драйвер, который позволит использовать Vue вместе с Vite.

```shell
npm install @vitejs/plugin-vue
```

## 1.6. Настройка Vite.

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

## 1.7. Инициализация Vue.

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
        <!-- Метатеги -->
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <meta http-equiv="Content-Security-Policy" content="upgrade-insecure-requests">

        <!-- SEO -->
        <meta name="keywords" content="ключевые слова, ключ, слово">
        <meta name="description" content="Описание лучшего сайта!">
        <meta name="author" content="разработчик">

        <!-- Open Graph -->
        <meta property="og:url" content="https://best-site-ever.ru/">
        <meta property="og:type" content="website" />
        <meta property="og:title" content="Заходи на сайт!">
        <meta property="og:description" content="Тут лучшие картинки и остальное...">
        <meta property="og:image" content="https://best-site-ever.ru/storage/img/logo.webp">
    </head>
    <body>
        <!-- элемент, в котором будет Vue приложение -->
        <div id="app"></div>
    
        <!-- импорт всего необходимого через Vite -->
        @vite('resources/js/app.js')
    </body>
</html>
```

## 1.8. Запуск приложения.

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
## 1.9. Vue Router.
Для начала установим сам Vue Router.

```shell
npm install vue-router
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
];
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

## 1.10. Настройка .htaccess на сервере для перенаправления на laravel.

Для этого нужно просто поместить следующий код внутри файла .htaccess в корне проекта:

```htaccess
<IfModule mod_rewrite.c>
    <IfModule mod_negotiation.c>
        Options -MultiViews -Indexes
    </IfModule>

    RewriteEngine On
    RewriteCond %{REQUEST_URI} !^public
    RewriteRule ^(.*)$ public/$1 [L]
</IfModule>
```

## 1.11. Как убрать build в url странице.

Для этого нужно просто назначить изначальный url для компиляции в router:

```diff
const router = createRouter({
+    history: createWebHistory(import.meta.env.VITE_BASE_URL || "/"),
   routes: [
      {
         path: "/",
         name: "Main",
         meta: { title: "Главная" },
         component: Main,
      },
   ],
});
```

А также настроить env переменную в .env:

```diff
+    VITE_BASE_URL="/"
```

# 2. Vue 3.
## 2.1. Отмотка страницы при переключении страниц.

Есть несколько способов сделать отмотку страницы:

- Добавление хука внутри объявления объекта **router**.

```js
const router = createRouter({
   history: createWebHistory(import.meta.env.VITE_BASE_URL || "/"),
   routes: [
      {
         path: "/",
         name: "Main",
         component: Main,
      }
   ],
   /* Отмотка страницы */
   scrollBehavior(to, from, savedPosition) {
      if (savedPosition) {
         return savedPosition;
      } else {
         return { top: 0 };
      }
   },
});
```

- Глобальный хук для прокрутки вверх после объявления **router**.

```js
// Глобальный хук для прокрутки вверх
router.afterEach(() => {
   window.scrollTo(0, 0);
});
```

- Использование хуков компонента.

```js
<template>
   <div>
      <!-- Ваш шаблон -->
   </div>
</template>

<script>
export default {
   name: "MyComponent",
   mounted() {
      window.scrollTo(0, 0);
   },
};
</script>
```

- Использование директивы.

```js
// directives/scrollToTop.js
export default {
   mounted(el) {
      el.addEventListener("click", () => {
         window.scrollTo(0, 0);
      });
   },
};
```

## 2.2. Изменение заголовка html страницы.

Чтобы менять заголовок страницы во время переключения страниц при помощи vue 3 нужно добавить в файл маршрутизации хук, который перехватывает процесс переключения страницы и меняет заголовок:

```js
// router/index.js
router.beforeEach((to, from, next) => {
   document.title = `${to.meta.title}`;
   next();
});
```

Теперь нам необходимо добавить содержимое заголовков, которе будет отображаться. Для этого нужно в каждый элемент массива путей добавлять следующее:

```diff
{
   path: "",
   name: "home",
+   meta: { title: "Главная" },
   component: () => import("../views/main/MainHome.vue"),
},
```

## 2.3. Установка axios, а также отлов ошибок в проекте.

Чтобы установить axios, необходимо ввести следующий код:

```shell
npm install axios
```

Для использования axios достаточно просто его импортировать в ваш модуль:

```js
import axios from 'axios';
```

Синтаксик стандартного вызова через axios выглядит следующим образом:

```js
axios({
   method: "post",
   url: `/api/index`,
   data: object
}).then((response) => {
   // ...
}).catch((error) => {
   // ...
}).finally(() => {
   // ...
});
```

Если вы хотите, чтобы все ошибки, которые возвращает catch отслеживались глобального из одного места, тогда можно сделать следующее:
- Добавить глобальный хук внутри файла main.js (который инициализирует vue). 
```js
import axios from "axios";

const app = createApp({
   extends: App,
   created() {
      // Отлов ошибок
      axios.interceptors.response.use(
         (response) => response,
         (error) => {
            console.log("ErrorHandler: " + error);

            switch (error.response.status) {
               // Не авторизован
               case 401:
                  this.$store.commit("logout");
                  break;
               }

         return Promise.reject(error);
      });
   },
});
```

- Сделать свой экземпляр наследник от axios. Для этого создаём отдельный файл .js, который будет нашим экземпляром: 
```js
import axios from 'axios';

// Создайте экземпляр Axios
import axios from "axios";
import store from "../store/index.js";

/* Создание axios */
const api = axios.create({
   // Базовый адрес
   baseURL: "/api",
   // Время запроса
   timeout: 10000,

   // Заголовки
   headers: {
      Accept: "application/json",
   },
});

// Перехватчик запросов (добавление токена)
api.interceptors.request.use((config) => {
   const token = localStorage.getItem("token");
   if (token) {
      config.headers.Authorization = `Bearer ${token}`;
   }

   return config;
});

/* Перехват ответа с сервера */
api.interceptors.response.use(
   (response) => {
      // Проверка успеха запроса
      if (!response.data.success) {
         if (response.data.message) {
            concole.log('Ошибка от сервера');
         } else {
            concole.log('Запрос не выполнен');	
         };

         return Promise.resolve(null);
      }

      if (response.data.debug) {
         concole.log('Запрос выполнен!');	
      }

      return Promise.resolve(response.data);
   },
   (error) => {
      // Нет ответа от сервера (нет интернета, таймаут)
      if (!error.response) {
         concole.log('Сервер не отвечает.');	

         return Promise.resolve(null);
      }

      const status = error.response.status;

      // Unauthorized
      if (status == 401) {
         store.commit("removeToken");
         return Promise.resolve(null);
      }

      // Forbidden
      if (status == 403) {
         concole.log('Нет доступа.');	

         return Promise.resolve(null);
      }

      // Not Found
      if (status == 404) {
         concole.log('Ресурс не найден!');	

         return Promise.resolve(null);
      }

      // Method Not Allowed
      if (status == 405) {
         concole.log('Метод не зарегистрирован!');	

         return Promise.resolve(null);
      }

      // Method Not Allowed, Internal Server Error
      if (status == 422 || status == 500) {
         if (error.response.data.errors) {
            for (let key in error.response.data.errors) {
               for (let item of error.response.data.errors[key]) {
	         concole.log('Ошибка: ' + item);	
               }
            }
         } else {
            concole.log(error.response.data);	
         }

         return Promise.resolve(null);
      }

      // Неизвестная ошибка
      concole.log('Не зарегистрированная ошибка.');

      return Promise.resolve(null);
   }
);

export default api;
```

После чего наш экземпляр можно будет использовать далее в проекте:

```js
import api from './api';

export default {
   methods: {
      fetchData() {
         api({
            method: "put",
            url: "/save", // /api уже добавлен в начало адреса
            data: {
               id: this.id,
               name: this.name,
               address: this.address,
            },
         })
         .then((response) => {
            if (response === null) {
               return;
            }

            try {
               // ...
            } catch (error) {
               // ...
            }
         })
         .finally(() => {
            this.disabled = false;
         });
      }
   }
}
```

# 3. Laravel 11.
## 3.1. Очистка и запуск конкретного файла миграции.

Для этого существует следующий синтаксик:

```php
php artisan migrate:refresh --path=database/migrations/1_create_employees_table.php
```
