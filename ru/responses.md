# HTTP-ответы

## Создание ответов

Все маршруты и контроллеры должны возвращать ответ, который будет отправлен в браузер пользователя. Imhotep предлагает несколько способов возврата ответа. Самый простой, это возврат строки, которую Imhotep автоматически преобразует в полноценный ответ:

```php
Route::get('/', function () {
    return 'Hello World';
});
```

Вы также можете возвращать массивы, Imhotep автоматически преобразует их в JSON ответы:

```php
Route::get('/', function () {
    return [1, 2, 3];
});
```

### Объект `Response`

Как правило, вместо возврата строк/массивов, вы будете возвращать экземпляр объекта `Imhotep\Http\Response` или `View`.

Возврат полного экземпляра `Response` позволяет настроить код состояния, HTTP заголовки:

```php
use Imhotep\Http\Response;

Route::get('/', function (Response $response) {
    return $response->content('Hello World')
                    ->code(200)
                    ->header('Content-Type', 'text/plain');
});
```

#### Добавление нескольких HTTP заголовков

Если необходимо добавить к ответу несколько заголовком, вы можете сделать это через метод `header` используя цепочку вызовов. Или вы можете использовать методы `headers` или `withHeaders` передав им ассоциативный массив заголовков:

```php
Route::get('/', function () {
    return response('Hello World', 200)->headers([
               'Content-Type' => 'text/plain',
               'X-Header-One' => 'Value',
               'X-Header-Two' => 'Value',
           ]);
});
```

#### Cache-Control

Imhotep содержит middleware `cache.headers`, используемый для быстрой установки заголовка Cache-Control. Директивы должны быть предоставлены с использованием эквивалента "snake case" соответствующей директивы управления кешем и должны быть разделены точкой с запятой. Если в списке директив указан etag, то MD5-хеш содержимого ответа будет автоматически установлен как идентификатор ETag:

```php
Route::middleware('cache.headers:public;max_age=2628000;etag')->group(function () {
    Route::get('/privacy', function () {
        // ...
    });

    Route::get('/terms', function () {
        // ...
    });
});
```

#### Cookies

Используя метод `cookie` вы можете добавить Cookies к вашему ответу, например:

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes
);
```

Метод `cookie` принимает такие же аргументы, как и встроенная в PHP функция [`setcookie`](https://www.php.net/manual/ru/function.setcookie.php):

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
```

Используя метод `queue` фасада `Cookie`, можно прикрепить куки к еще не созданному ответу, и перед отправкой ответа в браузер, они автоматически будут добавлены к ответу:

```php
use Imhotep\Facades\Cookie;

Cookie::queue('name', 'value', $minutes);
```

Также, вы можете создать экземпляр объекта `Imhotep\Http\Cookie`, который может быть добавлен к ответу позже. Для создания экземпляра `Imhotep\Http\Cookie` вы можете использовать глобальный помощник `cookie`. 

```php
$cookie = cookie('name', 'value', $minutes);

return response('Hello World')->cookie($cookie);
```

#### Удаление cookie
Для удаления cookie, используйте метод `withoutCookie`:

```php
return response('Hello World')->withoutCookie('name');
```

Если экземпляр ответа еще не был создан, вы можете воспользоваться методом `expire` фасада `Cookie` для обнуления срока действия куки:

```php
Cookie::expire('name');
```

## Перенаправления

## Типы ответов

## Макросы