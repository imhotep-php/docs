# Логирование

## Введение

Для отслеживания всех внутренних процессов работы приложения, Imhotep поддерживает службу логирования, которая умеет записывать сообщения в файлы, системный журнал или во внешние сервисы (например: Telegram).

Логирование основано на каналах. Каждый канал представляет определенный способ записи логов. Сообщения могут быть записаны в несколько каналов в зависимости от их важности.

## Конфигурирование

Все параметры логирования приложения расположены в файле `config/logging.php`. В данном файле находятся настройки распространенных каналов логирования, изучите параметры каждого канала.

По умолчанию, Imhotep использует канал `stack`, который объединяет несколько каналов логирования в один.

### Настройка имени канала

По умолчанию, имя канала соответствует текущей среде `production` или `local`. Для изменения имени канала, добавьте параметр `name` в настройки канала:

```php
'stack' => [
    'driver' => 'stack',
    'name' => 'channel-name',
    'channels' => ['single','telegram'],
],
```

## Логирование deprecations предупреждений

PHP, Imhotep и другие библиотеки часто уведомляют о том, что некоторые функции устарели и будут удалены в будущей версии. Для регистрации этих предупреждений, по умолчанию задан канал `deprecations` в конфигурационном файле приложения `config/logging.php`:

```php
'deprecations' => env('LOG_DEPRECATIONS_CHANNEL', 'null'),

'channels' => [
    // ...
    'deprecations' => [
        'driver' => 'single',
        'path' => storage_path('logs/deprecations.log'),
    ],
    // ...
]
```

## Запись сообщений в журнал

Imhotep позволяет логировать сообщения с помощью фасада Log. Приложение обеспечивает восемь уровней ведения, определенных в спецификации RFC 5424 specification: emergency, alert, critical, error, warning, notice, info, и debug.

```php
use Imhotep\Facades\Log;

Log::emergency($message);
Log::alert($message);
Log::critical($message);
Log::error($message);
Log::warning($message);
Log::notice($message);
Log::info($message);
Log::debug($message);
```

Можно вызвать любой из этих методов, чтобы записать сообщение для соответствующего уровня. По умолчанию сообщение будет записано в канал по умолчанию, указанный в конфигурации logging:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Imhotep\Facades\Log;

class UserController extends Controller
{
    public function show($id)
    {
        Log::info('Showing the user profile for user: '.$id);

        // ...
    }
}
```

### Контекстная информация

Методам логирования может быть передан массив контекстных данных. Эти данные будут отображены в каждой записи лога:

```php
Log::info('User failed to login.', ['id' => $id]);
```

Иногда может потребоваться указать дополнительную контекстную информацию, которая должна быть добавлена во все записи конкретного канала. Например, идентификатор запроса, связанный с каждым входящим запросом к приложению. Для этого используйте метод `withContext` фасада `Log`:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Imhotep\Facades\Log;
use Imhotep\Support\Str;

class AssignRequestId
{
    public function handle($request, Closure $next)
    {
        $requestId = Str::uuid();
        
        // Добавление информации в канал по умолчанию
        Log::withContext([
            'request-id' => $requestId
        ]);
        
        // Добавление информации в telegram канал
        Log::channel('telegram')->withContext([
            'request-id' => $requestId
        ]);

        return $next($request)->header('Request-Id', $requestId);
    }
}
```

Если необходимо добавить контекстную информацию во все созданные каналы и которые будут созданы позднее, используйте метод `Log::shareContext`. Данный метод предпочтительней вызывать из метода `boot` сервис-провайдера приложения:

```php
use Imhotep\Facades\Log;
use Imhotep\Support\Str;
 
class AppServiceProvider
{
    public function boot(): void
    {
        Log::shareContext([
            'some-id' => Str::uuid(),
        ]);
    }
}
```

