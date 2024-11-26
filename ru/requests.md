# HTTP-запросы

Класс `Imhotep\Http\Request` позволяет взаимодействовать с текущим HTTP-запросом, обрабатываемым вашим приложением, а также извлекать входные данные, cookies и файлы отправленные вместе с запросом.

## Доступ к запросу
Для получения экземпляра текущего HTTP-запроса, объявите класс Imhotep\Http\Request в методе контроллера. Экземпляр текущего запроса будет автоматически внедрен контейнером.

```php
namespace App\Http\Controllers;

use Imhotep\Http\Request;

class UserController extends Controller
{
    public function store(Request $request)
    {
        $method = $request->method();
    }
}
```

## Стандарт PSR-7
HTTP-запросы Imhotep реализуют интерфейс [PSR-7](https://www.php-fig.org/psr/psr-7/). Поэтому, вы можете объявить тип интерфейса, чтобы получить экземпляр текущего http-запроса:

```php
use Psr\Http\Message\ServerRequestInterface;

Route::get('/', function (ServerRequestInterface $request) {
    // ...
});
```

## Стройка запроса

Для получения HTTP-метода запроса используйте метод `method`. Также, можно воспользоваться методом `isMethod` для проверки на указанные HTTP-методы:

```php
$httpMethod = $request->method();
// или
$httpMethod = $request->getMethod();

// Вариант 1
if ($request->isMethod('HEAD')) {
    // ...
}

// Вариант 2
if ($request->isMethod('PUT','PATCH')) {
    // ...
}

// Вариант 3
if ($request->isMethod(['PUT','PATCH'])) {
    // ...
}
```

Если необходимо проверить, что запрос был запрошен через защищенный протокол, воспользуйтесь методом `secure` или `scheme`:

```php
if ($request->secure()) {
    // Действия для https запроса
}

if ($request->scheme() === 'https') {
    // ...
}
```

Для получения других данных о запросе, воспользуйтесь следующими функциями:

```php
// Request: https://example.com/user/profile?hidden=notifications

$root = $request->root(); // return 'https://example.com'
$host = $request->host(); // return 'example.com'
$port = $request->port(); // return '80'
$path = $request->path(); // return '/user/profile'
$queryString = $request->queryString(); // return 'hidden=notifications'
$uri = $request->uri(); // return '/user/profile?hidden=notifications'
```

### Получение URL-адреса запроса

Для получения URL-адреса запроса используйте метод `url`, `fullUrl` или `fullUrlWithQuery`. Метод url вернет URL без строки запроса, метод fullurl вернет URL адрес вместе со строкой запроса, а метод `fullUrlWithQuery` вернет URL запрос и добавит к текущей строке запроса данные из переданного массива:

```php
// Request: https://example.com/user/profile?hidden=notifications

// $url === 'https://example.com/user/profile'
$url = $request->url(); 

// $url === 'https://example.com/user/profile?hidden=notifications'
$url = $request->fullUrl();
$url = $request->url(true); // Альтернативный вариант

// $url === 'https://example.com/user/profile?hidden=notifications&from=register'
$url = $request->fullUrlWithQuery(['from' => 'register']);
$url = $request->url(['from' => 'register']); // Альтернативный вариант

// $url === 'https://example.com/user/profile?hidden=none'
$url = $request->fullUrlWithQuery(['hidden' => 'none']);
$url = $request->url(['from' => 'register']); // Альтернативный вариант
```

## Заголовки

Метод `headers` возвращает массив всех заголовков запроса. Если первым параметром передать имя заголовка, то будет возвращено значение запрошенного заголовка, иначе будет возвращено значение по умолчанию, переданное вторым аргументом:

```php
use Imhotep\Http\Request;

Route::get('/', function (Request $request) {
    // Получение списка всех заголовков запроса
    $headers = $request->headers();
    
    // Получение значения указанного заголовка
    $value = $request->headers('X-Header-Name');
    
    // Получить значение указанного заголовка,
    // иначе вернуть значение по умолчанию
    $value = $request->headers('X-Header-Name', 'default-value');
    
    $value = $request->headers('X-Header-Name', function () {
        return 'default-value';
    });
    
    // Альтернативный вариант, через метод header(key, [default])
    $value = $request->header('X-Header-Name', 'default-value');
    
    $value = $request->header('X-Header-Name', function () {
        return 'default-value';
    });
});
```

Используя метод `hasHeader` можно определить, содержит ли запрос заданный заголовок:

```php
if ($request->hasHeader('X-Header-Name')) {
    // ...
}
```

Для удобства, фреймворк поддерживает методы, позволяющие быстро получить распространенные данные из заголовков: 

* Метод `ip` - возвращает текущий IP-адрес пользователя.
* Метод `userAgent` - возвращает текущий User-Agent пользователя.
* Метод `ajax` - позволяет определить, является ли текущий запрос Ajax вызовом.
* Метод `pajax` - позволяет определить, является ли текущий запрос PAjax вызовом.
* Метод `prefetch` - позволяет определить, является ли текущий запрос предварительным.

### Согласование содержимого

Imhotep поддерживает проверку типов через заголовок `Accept`. Для получения списка всех типов контента доступных запросу, используйте метод `getAcceptableTypes`:

```php
$contentTypes = $request->getAcceptableTypes();
```

Метод `accepts` принимает массив типов контента и возвращает `true`, если хотя бы один из них поддерживается запросом. Иначе будет возвращено `false`:

```php
// Вариант 1
if ($request->accepts('text/html')) {
    // ...
}

// Вариант 2
if ($request->accepts(['text/html', 'application/json'])) {
    // ...
}

// Вариант 3
if ($request->accepts('text/html', 'application/json')) {
    // ...
}
```

Большинство приложений возвращают ответ в виде HTML или JSON, используйте метод `expectsJson`, чтобы быстро определить, ожидает ли входящий запрос JSON-ответ:

```php
if ($request->expectsJson()) {
    // ...
}
```

## Входные данные
Чтобы получить все данные входящего запроса в виде массива, используйте метод `all` или `input`. Этот метод можно использовать для всех типов запросов (HTTP-form, XHR...).

```php
$data = $request->all();

$data = $request->input();
```

### Получение данных по названию

Используя метод `input` можно получить значение любого поля данных их входящего запроса, не зависимо от типа HTTP-метода.

```php
$name = $request->input('name');
```

Данный метод в качестве второго аргумента принимает значение по умолчанию, которое будет возвращено в случае отсутствия поля во входящем запросе:

```php
$name = $request->input('name', 'Guest');
```

При работе с формами, содержащие массив входных данных, используйте «точечную» нотацию для доступа к элементам массива:

```php
$name = $request->input('products.0.name');

$names = $request->input('products.*.name');
```

### Получение данных из строки запроса

В то время как метод `input` извлекает все типы данных из запроса, то метод `query` извлекает данные только из строки запроса:

```php
$id = $request->query('id');
```

Если запрошенное значение в строке запроса отсутствует, метод возвратить значение переданное вторым аргументом:

```php
$id = $request->query('id', -1);
```

При вызове метода без аргументов, будет возвращен массив всех значений строки запроса:

```php
$query = $request->query();
```

### Получение значений JSON-содержимого

При отправке запросов JSON в ваше приложение, вы можете получить доступ к данным JSON с помощью метода input, если заголовок запроса Content-Type установлен как application/json. Вы также можете использовать «точечную» нотацию для извлечения значений, вложенных в JSON-массивы:

```php
$name = $request->input('user.name');
```

### Получение типизированных данных

Если необходимо получить значение и привести их к определенному типу данных, можно воспользоваться следующими методами:

```php
$name = $request->string('name');
$name = $request->str('name'); // Короткий вариант

$age = $request->integer('age');
$age = $request->int('age'); // Короткий вариант

$height = $request->float('height');

$archived = $request->boolean('archived');
$archived = $request->bool('archived'); // Короткий вариант
```

Каждый из этих методов принимает в качестве второго аргумента значение по умолчанию, которое будет возвращено если значение не будет найдено:

```php
$name = $request->str('name', 'Guest');

$age = $request->int('age', '21');

$height = $request->float('height', 179.5);

$archived = $request->bool('archived', true);
```

> Метод boolean возвращает true для 1, true, и строковых «1», «true», «on» и «yes». Все остальные значения вернут false.

### Получение данных через динамические свойства
Для получения доступа к поступившим от пользователя данных, можно использовать динамические свойства экземпляра `Imhotep\Http\Request`. Например, если запрос содержит поле `name`, то доступ можно получить следующим образом:

```php
$name = $request->name;
```

При использовании динамических свойств, сначала будет искаться значение параметра в информационной части данных запроса. Если его нет, то фреймворк будет искать поле в соответствующих параметрах маршрута.

### Частичное получение данных

Если необходимо получить подмножество входных данных, используйте метод `only`:

```php
$input = $request->only(['login', 'password']);

// Альтернативный вариант
$input = $request->only('login', 'password');
```

> Метод `only` возвращает пары ключ/значение, только тех данных, которые присутствуют в запросе.

## Обрезание и нормализаций входных данных

По умолчанию, Imhotep включает глобальные middleware `App\Http\Middleware\RequestTrimStrings` и `App\Http\Middleware\RequestEmptyStringsToNull`. Middleware `RequestTrimStrings` будет автоматически обрезать все входящие строковые поля запроса, а `RequestEmptyStringsToNull` конвертировать любые пустые строковые поля в null.

## Cookies

Для получения cookie данных из запроса, используйте метод `cookie` из экземпляра класса `Imhotep\Http\Request`:

```php
$token = $request->cookie('token');
```

Все значения cookie созданные Imhotep автоматически зашифрованы `APP_KEY` расположенный в конфигурационном файле `.env` и в случае изменения значений cookie на стороне клиента, эти значения станут недействительными. Для отключения шифрования cookie, удалите middleware `App\Http\Middleware\CookieEncryption` из массива `$middleware` в файле `app\Http\Kernel.php`.

## Файлы

### Получение загруженных файлов

Для получения загруженных файлов из экземпляра `Imhotep\Http\Request`, используйте метод `file` или динамические свойства. Метод `file` возвращает экземпляр класса `Imhotep\Http\UploadedFile`, который расширяет класс `SplFileInfo` и содержит различные методы для взаимодействия с файлом.

```php
$file = $request->file('photo');

$file = $request->photo;
```

Используя метод `hasFile`, можно определить наличие загруженного файла в запросе:

```php
if ($request->hasFile('photo')) {
    // ...
}
```

Для валидации загруженного файла, можно использовать методы `isValid` или `isValidImage`:

```php
if ($request->file('file')->isValid()) {
    // ...
}

if ($request->file('photo')->isValidImage()) {
    // ...
}
```

### Сохранение загруженных файлов

Для сохранения загруженного файла воспользуйтесь методом `store`, который перемещает файл на один из дисков (локальный или S3), настроенных в файловом хранилище.

Метод `store` первым аргументом принимаем путь, по которому файл должен храниться относительного настроенного каталога файловой системы. Этот путь не должен содержать имя файла, поскольку в качестве имени будет создан уникальный идентификатор. Второй аргумент принимает название диска, который следует использовать для сохранения файла. Метод вернет путь к файлу относительно корня диска:

```php
$path = $request->photo->store('images');

$path = $request->photo->store('images', 's3');
```

Если необходимо указать собственное имя файла, используйте метод `storeAs`, который принимает в качестве аргументов путь, имя файл и имя диска:

```php
$path = $request->photo->store('images', 'filename.jpg');

$path = $request->photo->store('images', 'filename.jpg', 's3');
```

> Для получения дополнительной информации о работе с файловой системе Imhotep, ознакомьтесь с полной документацией.






Метод `is` проверяет соответствие пути входящему запроса по шаблону:

```php
if ($request->is('admin/*')) {
    //
}
```

Используя метод routeIs, вы можете определить, соответствует ли входящий запрос именованному маршруту:

```php
if ($request->routeIs('admin.*')) {
    //
}


$request->is();
$request->routeIs();

$request->has('name')
$request->has(['name', 'email'])
$request->whenHas('name', function ($input) { });
$request->whenHas('name', function ($input) {
    // Значение "имя" присутствует...
}, function () {
    // Значение "имя" отсутствует...
});
if ($request->hasAny(['name', 'email'])) {
    //
}
if ($request->filled('name')) {
    //
}
$request->whenFilled('name', function ($input) {
    //
});
$request->whenFilled('name', function ($input) {
    // Значение "имя" заполнено ...
}, function () {
    // Значение "имя" не заполнено...
});

if ($request->missing('name')) {
    //
}

// Кратковременное сохранение входных данных в сессии
$request->flash();
$request->flashOnly(['username', 'email']);
$request->flashExcept('password');

// Кратковременное сохранение при перенаправлении
redirect('form')->withInput();
redirect()->route('user.create')->withInput();
redirect('form')->withInput(
    $request->except('password')
);

// Получение данных прошлого запроса
$username = $request->old('username');
```