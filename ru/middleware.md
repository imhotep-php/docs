# Middleware

Middleware предоставляют удобный механизм предварительной обработки входящего HTTP-запроса.

## Создание middleware

Для создания нового middleware, выполните консольную команду:

```shell
php imhotep make:middleware IsHttpsRequest
```

Данная команда, создаст новый класс `EnsureHttpsRequest` в папке `app/Http/Middleware`. В данном middleware будем проверять, что входящий запрос был передан по шифрованному https протоколу, иначе перенаправить его:

```php
<?php
 
namespace App\Http\Middleware;
 
class EnsureHttpsRequest
{
    public function handle($request, \Closure $next)
    {
        if (! $request->secure()) {
            return redirect()->secure($request->uri());
        }
 
        return $next($request);
    }
}
```

Как видим выше, если запрос не является безопасным, то middleware возвращает HTTP-перенаправление, иначе запрос будет передан следующему middleware или в контроллер. Чтобы запрос был передан дальше, необходимо вернуть замыкание `$next` с параметром `$request`.

> Все middleware извлекаются через контейнер, поэтому вы можете объявить необходимые вам зависимости в конструкторе вашего класса middleware.


### Middleware & Responses

Без сомнений, middleware могут выполнять задачи до и после передачи запроса в контроллер. Например, следующий middleware выполнит задачу до передачи запроса в контроллер:

```debug
<?php
 
namespace App\Http\Middleware;
 
class BeforeMiddleware
{
    public function handle($request, \Closure $next)
    {
        // Необходимое действие
 
        return $next($request);
    }
}
```

Однако, этот middleware выполнит задачу после обработки входящего запроса контроллером:

```php
<?php
 
namespace App\Http\Middleware;
 
class AfterMiddleware
{
    public function handle($request, \Closure $next)
    {
        $response = $next($request);
        
        // Необходимое действие
 
        return $response;
    }
}
```

Также, вы можете выполнить задачи до и после обработки запроса в одном middleware классе:

```php
<?php
 
namespace App\Http\Middleware;
 
class Middleware
{
    public function handle($request, \Closure $next)
    {
        // Действия, необходимые выполнить до передачи запроса в контроллер
      
        $response = $next($request);
        
        // Действия, необходимые выполнить после обработки запроса в контроллере
 
        return $response;
    }
}
```

## Регистрация middleware

### Глобальные middleware
Если необходимо запускать middleware для всех входящих HTTP-запросов, добавьте класс middleware в список свойства `$middleware` в файле `app/Http/Kernel.php`.

### Назначение middleware маршрутам
Если хотите назначать middleware к определенным маршрутам, тогда следует сначала зарегистрировать псевдоним в файле `app/Http/Kernel.php`. По умолчанию, свойство `$routeMiddleware` содержит middleware которые включены в состав Imhotep. Вы можете добавить свой собственный middleware в конец списка и назначить ему любой псевдоним.

```php
// Внутри класса app/Http/Kernel.php
 
protected $routeMiddleware = [
    // ...
    'https' => \App\Http\Middleware\EnsureHttpsRequest::class
];
```

После того как middleware был зарегистрирован в HTTP-ядре, вы можете использовать метод `middleware` для привязки middleware к маршруту.

```php
Route::get('/profile', function () {
    // ...
})->middleware('auth');
```

Вы можете назначить несколько middleware, передав в метод `middleware` массив значений.

```php
Route::get('/', function () {
    // ...
})->middleware(['first', 'second']);
```

Так же, вы можете назначить middleware, передав полное имя класса:

```php
use \App\Http\Middleware\EnsureHttpsRequest;

Route::get('/', function () {
    // ...
})->middleware(EnsureHttpsRequest::class);
```

Или использовать замыкание:

```php
Route::get('/', function () {
    // ...
})->middleware(function ($request, \Closure $next) {
    // Необходимое действие
    return $next($request);
});
```

#### Исключение middleware
При назначении middleware группе маршрутов, может потребоваться запретить применение middleware к одному из маршрутов. Это можно сделать с помощью метода `withoutMiddleware`:

```php
use \App\Http\Middleware\EnsureHttpsRequest;

Route::middleware([EnsureHttpsRequest::class])->group(function () {
    Route::get('/', function () {
        // ...
    })->withoutMiddleware([EnsureHttpsRequest::class]);

    Route::get('/profile', function () {
        // ...
    });
});
```

Вы также можете исключить middleware для всей группы маршрутов:

```php
use \App\Http\Middleware\EnsureHttpsRequest;

Route::withoutMiddleware([EnsureHttpsRequest::class])->group(function () {
    Route::get('/', function () {
        // ...
    })

    Route::get('/profile', function () {
        // ...
    });
});
```

Метод withoutMiddleware только middleware напрямую привязанные к маршруту и не применим к middleware из глобального списка.


### Группы middleware

По желанию, можно сгруппировать несколько middleware под одним ключом, чтобы упростить их назначение маршрутам. Это можно сделать используя массив `$middlewareGroups` в файле `app/Http/Kernel.php`.

### Сортировка middleware
Иногда, может понадобиться, чтобы middleware выполнялись в определенном порядке, но вы не можете контролировать их порядок, когда они назначены маршруту. В этом случае укажите приоритет middleware, перечислив их в заданном порядке в массиве `$middlewarePriority` файла `app/Http/Kernel.php`.

## Middleware параметры

Middleware могут получать дополнительные параметры. Например, если приложению требуется проверить, что аутентифицированный пользовать имеет определенную «роль» для выполнения заданного действия, тогда вы можете создать middleware, например `EnsureUserHasRole`, который получит имя роли в качестве дополнительного аргумента.

Дополнительные параметры middleware будут переданы после аргумента `$next`:

```php
<?php

namespace App\Http\Middleware;

class EnsureUserHasRole
{
    public function handle($request, \Closure $next, $role)
    {
        if (! $request->user()->hasRole($role)) {
            // Перенаправление ...
        }

        return $next($request);
    }

}
```

Параметры middleware можно указать при определении маршрута, разделим имя middleware и параметры символом `:`. Несколько параметров следует разделять запятыми:

```php
Route::put('/post/{id}', function ($id) {
    // ...
})->middleware('role:editor');
```

## Завершающий middleware

Иногда может потребоваться выполнить некоторую работу после отправки HTTP-ответа в браузер. Определите метод `terminate` в своем middleware, и он будет вызван после отправки ответа в браузер, при условии, что ваш веб-сервер использует FastCGI:

```php
<?php

namespace App\Http\Middleware;

class TerminatingMiddleware
{
    public function handle($request, \Closure $next)
    {
        return $next($request);
    }

    public function terminate($request, $response)
    {
        // ...
    }
}
```

После создания завершающего middleware, его необходимо добавить в глобальный список middleware или назначить необходимым маршрутам.

При вызове middleware метода `terminate`, Imhotep каждый раз извлекает новый экземпляр middleware. Если вы хотите использовать один и тот же экземпляр middleware при вызове метода `handle` и `terminate`, то зарегистрируйте middleware в контейнере, используя метод `singleton`. Обычно это делается в методе `register` вашего `AppServiceProvider`.

```php
// Внутри файла app/Providers/AppServiceProvider.php

use App\Http\Middleware\TerminatingMiddleware;

public function register()
{
    $this->app->singleton(TerminatingMiddleware::class);
}
```