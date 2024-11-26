# Service Providers

- Введение
- Создание провайдера
  - Метод register
  - Метод boot
- Регистрация провайдеров
- Отложенные провайдеры

## Введение
Service Providers, далее «сервис-провайдер» - это центральная точка начальной загрузки и конфигурирования всего приложения Imhotep. Сервис-провайдеры регистрируют и связывают компоненты с контейнером, настраивают event listeners, middleware, routes и т.д..

Откройте файл `config/app.php` и найдите массив `'providers'` - это всё классы сервис-провайдеры, которые будут загружены приложением. По умолчанию, в данном массив перечислены основные сервис-провайдеры Imhotep, которые загружают события, кеширование, базу данных, маршрутизацию и другое. Многие из них являются "отложенными" - т.е. они будут загружены только при первом обращении к их службам.

В данной документации вы узнаете, как создавать сервис-провайдеры и регистрировать их в приложении Imhotep.

> Для более глубокого понимания работы Imhotep, рекомендуем изучить жизненный цикл запроса.

## Создание провайдера
Все сервис-провайдеры расширяют класс `Imhotep\Framework\Providers\ServiceProvider`. Большинство из них содержат методы `register` и `boot`. Для создания провайдера используйте консольную команду:

```shell
php imhotep make:provider RiakServiceProvider
```

### Метод register
В методе `register` следует только связывать классы с контейнером. Никогда не пытайтесь зарегистрировать события, маршруты или что-то другое - это может привести к тому, что вы обратитесь к подсистеме, чей сервис-провайдер еще не загружен. В сервис-провайдере у нас всегда есть доступ к контейнеру, через переменную `$app`:

```php
namespace App\Providers;

use App\Services\Riak\Connection;
use Imhotep\Framework\Providers\ServiceProvider;

class RiakServiceProvider extends ServiceProvider
{
    public function register()
    {
        $this->app->singleton(Connection::class, function ($app) {
            return new Connection(config('riak'));
        });
    }
}
```

Данный сервис-провайдер реализовывает только метод `register`, указывая какую именно реализацию `App\Services\Riak\Connection` применять в приложении при извлечении через контейнер.

#### Свойства `bindings`, `singletons` и `aliases`
Если сервис-провайдер регистрирует простые связывания, вы можете использовать свойства `bindings`, `singletons` и `aliases`, вместо ручной регистрации каждого связывания в контейнере. Когда сервис-провайдер загружается приложением, он автоматический проверяет эти свойства и связывает их в контейнере.

```php
namespace App\Providers;

use App\Contacts\ProxyProvider;
use App\Contacts\MailSender;
use App\Services\HostingProxyProvider;
use App\Services\HostingMailSender;
use Imhotep\Framework\Providers\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public array $bindings = [
        ProxyProvider::class => HostingProxyProvider::class,
    ];
    
    public array $singletons = [
        MailSender::class => HostingMailSender::class,
        ProxyProvider::class => HostingProxyProvider::class,
    ];
    
    public array $aliases = [
        'mail' => MailSender::class,
    ];
}
```

### Метод boot
Метод `boot` вызывает после регистрации всех сервис-провайдеров, что позволяет иметь доступ ко всех другим службам приложения. В рамках данного метода, вы можете конфигурировать сервисы, создавать маршруты и многое другое, а благодаря контейнеру, вы можете автоматически внедрять любые необходимые зависимости:

```php
namespace App\Providers;

use Imhotep\Framework\Providers\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(MailSender $mail)
    {
        $mail->setConfig([/*...*/]);
        
        Route::post('/mail/sender', function () {
           // ... 
        });
    }
}
```

## Регистрация провайдеров
Все сервис-провайдеры регистрируются в конфигурационном файле `config/app.php`. В массиве `providers`, по умолчанию перечислены основные сервис-провайдеры приложения, которые загружают базовые компоненты Imhotep.

Для регистрации нового сервис-провайдера, добавьте его в массив:

```php
'providers' => [
    // Другие сервис-провайдеры
    
    App\Providers\ComposerServiceProvider::class,
],
```

## Отложенные провайдеры

Если ваш сервис-провайдер регистрирует только связывания в контейнере, вы можете отложить регистрацию, до тех пор, пока одно из зарегистрированных связываний не понадобится. Отсрочка загрузки такого сервис-провайдера повышает производительность приложения, так как он не загружается из файловой системы при каждом запросе.

Чтобы отложить загрузку сервис-провайдера, реализуйте интерфейс `Imhotep\Framework\Providers\DeferrableProvider` и реализуйте метод `provides`. Метод `provides` должен вернуть связывания для контейнера, регистрируемые данным сервис-провайдером.

```php
namespace App\Providers;

use App\Services\Riak\Connection;
use Imhotep\Framework\Providers\ServiceProvider;
use Imhotep\Framework\Providers\DeferrableProvider;

class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
{
    public function register()
    {
        $this->app->singleton(Connection::class, function ($app) {
            return new Connection(config('riak'));
        });
    }
    
    public function provides()
    {
        return [Connection::class];
    }
}
```