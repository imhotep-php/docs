<a name="container"></a>
# Service Container

<!-- NavigationBegin -->
- [Введение](#introduction)
- [Базовое распознавание](#basic)
- [Когда использовать Container](#when-use)
- [Связывание (Binding)](#binding)
  - [Простое связывание](#binding-simple)
  - [Связывание одиночек](#binding-singlenton)
  - [Связывание одиночек с ограничением](#binding-singleton-scoped)
  - [Связывание экземпляров](#binding-instance)
  - [Связывание интерфейсов с реализацией](#binding-interface-with-instance)
  - [Создание меток](#tags)
  - [Контекстное связывание](#contextual)
  - [Связывание переменных](#binding-primitives)
  - [Связывание вариативных типов](#binding-variadic)
  - [Расширение привязок](#binding-extend)
- [Извлечение](#resolve)
  - [Метод make](#make-method)
  - [Автоматическое внедрение зависимостей](#injection)
- [Вызов метода и внедрение](#call-method-and-injection)
- [События контейнера](#events)
- [PSR-11](#psr-11)
<!-- NavigationEnd -->

<a name="introduction"></a>
## Введение

Service Container, далее «контейнер» - это мощный инструмент для управления зависимостями классов и внедрения зависимости (dependency injection). Внедрение зависимостей - это эффективная идея, согласно которой: зависимости класса "injected" в класс через конструктор или через "setter" методы.

Давайте посмотрим на простой пример:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Repositories\UserRepository;
use App\Models\User;

class UserController extends Controller
{
    protected $users;
 
    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }
 
    public function show($id)
    {
        $user = $this->users->find($id);
 
        return view('user.profile', ['user' => $user]);
    }
}
```

В данном пример, `UserController` требуется извлечение пользователей из источника данных. Теперь, нам нужно внедрить службу, способную извлекать пользователей. В данном примере, `UserRepository` может использовать Calypte для извлечения пользователей из базы данных. Однако, мы можем использовать "mock" или фиктивных реализаций `UserRepository` для тестирования приложения.

Глубокое понимание Service Container позволит создавать большие и мощьные приложения, а также понимать само ядро Imhotep.

<a name="basic"></a>
## Базовое распознавание

Если класс не имеет зависимостей или зависит только от другого конкретного класса (не интерфейса), контейнеру не требуется дополнительная информация для его распознавания. Например, в файле `routes/web.php` вы можете написать следующий код:

```php
class Service {
    // 
}

Route::get('/', function (Service $service) {
    die(get_class($service));
});
```

В этом примере, когда вы переходите по маршруту `/`, приложение автоматически распознает класс `Service` и внедрит его в обработчик маршрута.

<a name="when-use"></a>
## Когда использовать Container
Благодаря нулевой конфигурации, вы часто будете указывать зависимости в routes, controllers, event listeners и в других местах, без взаимодействия с service container вручную. Например, вы можете написать объект `Imhotep\Http\Request` в объявлении обработчика маршрута, что бы легко получить доступ к текущему запросу:

```php
use Imhotep\Http\Request;

Route::get('/', function (Request $request) {
    // ...
});
```

Несмотря на то, что в данном случае нам никогда не потребуется взаимодействовать с Container, он управляет внедрением этих зависимостей за кулисами.

Благодаря автоматическому внедрению зависимостей, вы можете создавать приложения без жесткой привязки классов между собой. Но при каких условиях нам придется вручную взаимодействовать в Container? Разберем две ситуации.

Во-первых, если вы пишете класс, реализующий интерфейс, и хотите указать этот тип интерфейса в конструкторе маршрута или класса, вам необходимо сообщить контейнеру как распознать этот интерфейс.

Во-вторых, если вы пишете пакет Imhotep, которым планируете поделиться с другими разработчиками, вам может потребоваться привязать службы вашего пакета к Container.

<a name="binding"></a>
## Связывание (Binding)

<a name="binding-simple"></a>
### Простое связывание
Почти все bindings в Container происходят во время регистрации service providers.

Внутри service provider вы легко можете получить доступ к Container через свойство `$this->app`. Вы можете зарегистрировать bindings используя метод `bind`, передав первым аргументом имя класса или интерфейса, а вторым замыкание возвращающие экземпляр данного класса.

```php
use App\Services\Wiki;
use App\Services\GithubParser;

$this->app->bind(Wiki:class, function($app) {
    return new Wiki($app->make(GithubParser::class));
});
```

Обратите внимание, в качестве первого аргумента `$app` в замыкание передается Container. Это позволяет распознавать связывания в зависимых объектах.

Как говорилось ранее, обычно вы будете взаимодействовать с Container внутри service providers. Однако, если вам потребуется сделать это за их пределами, вы можете использовать `App` фасад:

```php
use App\Services\Wiki;
use App\Services\GithubParser;

App::bind(Wiki:class, function($app) {
    // ...
});
```

> **Примечание**  
> Нет необходимости в связывании классов внутри Container, если они не зависят от других классов/интерфейсов. Container способен автоматически создать эти объекты с помощью reflection.

<a name="binding-singlenton"></a>
### Связывание одиночек
Методы `singleton` связывает класс/интерфейс в Container, экземпляр которого должен быть создан только один раз. При повторном обращении к этому классу в Container, будет возвращен ранее созданный экземпляр объекта.

```php
use App\Services\Wiki;
use App\Services\GithubParser;

$this->app->singleton(Wiki:class, function($app) {
    return new Wiki($app->make(GithubParser::class));
});
```

<a name="binding-singleton-scoped"></a>
### Связывание одиночек с ограничением
Метод `scoped` похож на `singleton` с разницей в том, что объект будет создан в рамках жизненного цикла запроса, после чего будет удален. Например, при обработке нового задания в очереди заданий.

```php
use App\Services\Wiki;
use App\Services\GithubParser;

$this->app->scoped(Wiki:class, function($app) {
    return new Wiki($app->make(GithubParser::class));
});
```

<a name="binding-instance"></a>
### Связывание экземпляров
Метод `instance` позволяет привязать существующий экземпляр объекта к конкретному классу/интерфейсу и во всех последующих вызовах класса/интерфеса, Container вернет данный экземпляр.

```php
use App\Services\Wiki;
use App\Services\GithubParser;

$wiki = new Wiki(new GithubParser());

$this->app->instance(Wiki:class, $wiki);
```

<a name="binding-interface-with-instance"></a>
### Связывание интерфейсов с реализацией
Возможность связывания интерфейса с конкретной его реализаций, позволяет создавать очень мощные и гибкие приложения. Например, у нас есть интерфейс `CacheStore` и реализация `FileCacheStore`, теперь мы можем связать их в Container следующим образом:

```php
use App\Contracts\CacheStore;
use App\Services\FileCacheStore;

$this->app->bind(CacheStore:class, FileCacheStore::class);
```

Это связывание сообщает Container, что он должен вернуть `FileCacheStore` каждый раз, когда требуется внедрить `CacheStore`. Теперь мы можем указать в конструкторе класса интерфейс `CacheStore`, который будет извлечен из Container:

```php
use App\Contracts\CacheStore;

class Cache
{
    protected CacheStore $store;
    
    public function __construct(CacheStore $store) {
        $this->store = $store;
    }
    
    // ...
}
```

<a name="tags"></a>
### Создание меток
С помощью метода `tag` вы можете создать набор объектов для привязки определенной категории. Например, если вы создаете систему фильтрации данных, которая получает вариативности различных реализаций интерфейса `Filter`. После регистрации реализаций интерфейса `Filter` вы можете назначить им метку.

```php
$this->app->bind(IntegerFilter::class, function () {
    // ...
});

$this->app->bind(StringFilter::class, function () {
    // ...
});

$this->app->tag([IntegerFilter::class, StringFilter::class], 'filters');
```

После того как тег был создан, вы можете легко получить все объекты вызвав метод `tagged`:

```php
$this->app->bind(Firewall::class, function ($app) {
    return new Firewall($app->tagged('filters'));
});
```

<a name="contextual"></a>
### Контекстное связывание
Иногда два разных класса с одним интерфейсом в конструкторе, могут потребовать внедрения разных реализаций в каждый класс. Например, два контроллера могут зависеть от разных реализаций интерфейса `CacheStore`:

```php
use App\Contracts\CacheStore;
use App\Services\RedisStore;
use App\Services\FileStore;

$this->app->when(AuthController::class)
          ->needs(CacheStore::class)
          ->give(function () {
              return new RedisStore();
          });

$this->app->when([PageController::class, NewsController::class])
          ->needs(CacheStore::class)
          ->give(function () {
              return new FileStore();
          });
```

<a name="binding-primitives"></a>
### Связывание переменных
Иногда может быть класс в конструкторе которого требуется внедрение переменной, например целое число. Вы можете использовать контекстное связывание для внедрения любого значения.

```php
$this->app->when(AuthController::class)
          ->needs('$variableName')
          ->give($value);
```

Если необходимо внедрить значение из конфигурационного файла, воспользуйтесь методом `giveConfig`:

```php
$this->app->when(AuthController::class)
          ->needs('$appName')
          ->giveConfig('app.name');
```

Иногда класс может зависеть от массива экземпляров, объединенных тегом. Используя `giveTagged` вы можете легко их внедрить:

```php
$this->app->when(AuthController::class)
          ->needs('$reports')
          ->giveTagged('reports');
```

<a name="binding-variadic"></a>
### Связывание вариативных типов
Иногда класс может зависеть от массива типизированных обектов переданные в качестве вариативного аргумента:

```php
class Firewall
{
    protected $logged;
    
    protected $filters;
    
    public function __constructor(Logger $logger, Filter ...$filter)
    {
        $this->logged = $logger;
        $this->filters = $filter;
    }
}
```
Используя контекстное связывание вы можете внедрить зависимость используя метод `give` с замыканием, которое возвращает массив объектов `Filter`.

```php
$this->app->when(Firewall::class)
          ->needs(Filter::class)
          ->give(function ($app) {
                return [
                    $app->make(NullFilter::class),
                    $app->make(StringFilter::class),
                    $app->make(IntegerFilter::class),
                ];
          });
```

Для упрощения вы можете передать просто массив имен классов который будут автоматически созданы каждый раз когда `Firewall` потребуются объекты `Filter`.

```php
$this->app->when(Firewall::class)
          ->needs(Filter::class)
          ->give([
              NullFilter::class,
              StringFilter::class,
              IntegerFilter::class
          ]);
```

Вы также можете использовать вариативный тег для внедрения зависимости:

```php
$this->app->tag([
                NullFilter::class,
                StringFilter::class,
                IntegerFilter::class
            ], 'filters');

$this->app->when(Firewall::class)
          ->needs(Filter::class)
          ->giveTagged('filters');
```

<a name="binding-extend"></a>
### Расширение привязок

Метод `extend` позволяет модифицировать объект. Например, когда объект создан, вы можете запустить дополнительный код для его декорирования или конфигурирования. Метод `extend` принимает замыкание, которое возвращает измененный объект. Замыкание получает объект и экземпляр контейнера:

```php
$this->app->extend(Firewall::class, function ($firewall, $app) {
    return new DecoratedFirewall($firewall);
});
```

<a name="resolve"></a>
## Извлечение

<a name="make-method"></a>
### Метод make

Используя метод `make` вы можете извлечь экземпляр объекта из контейнера. Метод `make` принимает имя класса или интерфейс, который вы хотите извлечь.

```php
use App\Service\Firewall;

$firewall = $this->app->make(Firewall::class);
```

Если конструктор класса зависит от дополнительных параметров, которые контейнер не может распознать автоматически, вы можете передать их вторым параметром как ассоциативный массив. Например, мы можем передать аргумент `id` в конструктор класса `Transitor`.

```php
use App\Service\Transitor;

$transitor = $this->app->make(Transitor::class, ['id' => 1]);
```

Если вы находитесь за пределами Service Provider и не имеете доступа к переменной `$app`, вы можете использовать фасад `App` или хелпер `app` для извлечения экземпляра класса из контейнера.

```php
use App\Service\Firewall;
use Imhotep\Facades\App;

$firewall = App::make(Firewall::class);
// or
$firewall = app(Firewall::class);
```

Если вы хотите, что бы экземпляр контейнера был внедрен в класс извлекаемый контейнером, вы можете указать `Imhotep\Container\Container` в конструкторе класса.

```php
use Imhotep\Container\Container;

class MyClass
{
    protected $container;
    
    public function __construct(Container $container)
    {
        $this->container = $container;
    }
}
```

<a name="injection"></a>
### Автоматическое внедрение зависимостей

Помимо того, что вы можете внедрить зависимости в конструкторе класса извлекаемого из контейнера, вы так же можете внедрить их в методе `handle` обработки маршрутизации.

Например, вы можете объявить `UserRepository` в конструкторе контроллера, а так же внедрить текущий запрос `Request` в метод `show` для обработчика маршрута.

```php
use App\Repositories\UserRepository;

class UserController
{
    protected $repository;
    
    public function __construct(UserRepository $repository)
    {
        $this->repository = $repository;
    }
    
    public function show(Request $request, $id)
    {
        // ...
    }
}
```

<a name="call-method-and-injection"></a>
### Вызов метода и внедрение

Иногда может потребоваться вызвать метод у объекта, позволив контейнеру автоматически внедрить зависимости этого метода. Например, рассмотрим следующий класс:

```php
use App\Repositories\UserRepository;

class UserReport
{
    public function generate(UserRepository $repository)
    {
        // ...
    }
}
```

Вы можете вызвать метод `generate` следующим способом:

```php
use App\UserReport;
use Imhotep\Facades\App;

$report = App::call([UserReport::class, 'generate']);
```

Метод `call` принимает любой вызываемый PHP-код. Он так же может использоваться для вызова замыкания с автоматическим внедрением зависимостей:

```php
use App\Repositories\UserRepository;
use Imhotep\Facades\App;

$report = App::call(function (UserRepository $repository) {
    // ...
});
```

<a name="events"></a>
## События контейнера
Контейнер инициирует событие каждый раз, когда извлекается объект. Вы можете прослушать это событие с помощью метода `resolving`, в переданное замыкание будет передан первым параметром извлекаемый объект, а вторым экземпляр контейнера. Это позволит вам установить дополнительные свойства объекту до того, как он будет передан получателю:

```php
use App\Repositories\UserRepository;

$this->app->resolving(UserRepository::class, function ($repository, $app) {
    // Вызывается, когда контейнер извлекает объект `UserRepository`
});

$this->app->resolving(function ($object, $app) {
    // Вызывается, когда контейнер извлекает объект любого типа
});
```

<a name="psr-11"></a>
## PSR-11
Контейнер служб Imhotep реализует интерфейс PSR-11. Поэтому, вы можете объявить тип интерфейса, что бы получить экземпляр контейнера Imhotep.

```php
use App\Controllers\UserController;

Route::get('/', function (ContainerInterface $container) {
    $controller = $container->get(UserController::class);
    
    // ...
});
```

Если контейнер не смог извлечь объект из-за отсутствия привязки будет выброшен экземпляр исключения `Psr\Container\NotFoundExceptionInterface`, если объект был привязан, но не может быть извлечен, будет выброшен экземпляр исключения `Psr\Container\ContainerExceptionInterface`.
