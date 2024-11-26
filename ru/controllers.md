# Контроллеры

Контроллеры необходимы для группировки связанной логики обработки запросов в одном классе.

## Создание контроллеров
По умолчанию, все контроллеры хранятся в каталоге `app/Http/Controllers`, где так же находится базовый класс `App\Http\Controllers\Controller`, включенный в Imhotep. Рассмотрим пример обычного контроллера:

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;

class UserController extends Controller
{
    public function show($id)
    {
        return "User-{$id}";
    }
}
```

Вы можете определить маршрут к этому методу контроллера следующим образом:

```php
use App\Http\Controllers\UserController;

Route::get('/user/{id}', [UserController::class, 'show']);
```

Когда входящий запрос совпадает с указанным URI маршрута, будет вызван метод show класса App\Http\Controllers\UserController, и параметры маршрута будут переданы методу.

> Контроллеры не требуют расширения базового класса. Однако у вас не будет доступа к удобным функциям, например middleware.

### Контроллеры одного действия
Если действие контроллера является сложным, вы можете посвятить целый класс этому единственному действию. Для этого вы можете определить в контроллере один метод `__invoke`:

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;

class ProvisionServer extends Controller
{
    public function __invoke()
    {
        // ...
    }
}
```

При регистрации маршрутов для контроллеров одиночного действия вам не нужно указывать метод контроллера. Вместо этого вы можете просто передать имя контроллера:

```php
use App\Http\Controllers\ProvisionServer;

Route::post('/server', ProvisionServer::class);
```

### Консольные команды

Для создания обычного контроллера через консоль, используйте следующую команду:

```shell
php imhotep make:controller UserController
```

Для создания одиночного контроллера:

```shell
php imhotep make:controller ProvisionServer --invokable
```

## Middleware и контроллеры
Используя метод `middleware` в конструкторе вашего контроллера, вы можете назначить middleware действиям контроллера:

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;

class UserController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth');
        $this->middleware('log')->only('index');
        $this->middleware('subscribed')->except('store');
    }
}
```

Вы также можете назначать middleware через замыкания. Это позволяет использовать уникальные middleware для конкретного контроллера, без создания отдельного целого класса middleware:

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;

class UserController extends Controller
{
    public function __construct()
    {
        $this->middleware(function ($request, $next) {
            // Необходимые действия
            return $next($request);
        });
    }
}
```

## Внедрение зависимостей и контроллеры

Контейнер Imhotep используется для извлечения всех контроллеров. Это позволяет объявлять любые зависимости необходимые вашему контроллеру в его конструкторе:

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Repositories\UserRepository;

class UserController extends Controller
{
    protected $users;

    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }
}
```

Вы также можете объявлять необходимые вам зависимости и в методах контроллера:

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;

class UserController extends Controller
{
    public function store(Request $request)
    {
        $name = $request->name;
    }
}
```

Если метод контроллера ожидает входные данные из параметров маршрута, укажите аргументы после другим зависимостей, например, если маршрут зарегистрирован так:

```php
use App\Http\Controllers\UserController;

Route::put('/user/{id}', [UserController::class, 'update']);
```

То контроллер будет выглядеть следующим образом:

```php
namespace App\Http\Controllers;

use Imhotep\Http\Request;

class UserController extends Controller
{
    public function update(Request $request, $userId)
    {
        // ...
    }
}
```