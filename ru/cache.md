<a name="cache"></a>
# Кэширование

<!-- NavigationBegin -->
- [Введение](#introduction)
- [Конфигурирование](#config)
    - [Подготовка драйвера database](#cache-table)
- [Взаимодействие](#interaction)
    - [Доступ к кэш-хранилищу](#store)
    - [Проверка наличия данных](#has)
    - [Получение данных](#get)
    - [Сохранение данных](#set)
    - [Сохранение массива элементов](#set-many)
    - [Increment / Decrement](#increment-decrement)
    - [Извлечь и сохранить](#remember)
    - [Удаление данных](#delete)
- [Создание своего драйвера](#create-driver)
<!-- NavigationEnd -->

<a name="introduction"></a>
## Введение

Кэширование — это процесс, который оптимизирует работу приложения и снижает нагрузку на сервер. Он достигается за счёт сохранения часто используемых данных в памяти, что позволяет быстро извлекать их при последующих запросах.

Imhotep предлагает разнообразные системы кэширования, что даёт вам возможность выбрать наиболее подходящую под ваши требования к приложению.

<a name="config"></a>
## Конфигурирование

Файл `config/cache.php` содержит все необходимые настройки системы кэширования. Откройте файл и изучите содержимое.

<a name="cache-table"></a>
### Подготовка драйвера database

Для работы драйвера кэша database необходима таблица в выбранной базе данных. По умолчанию миграция уже включена в Imhotep и находится в файле `0001_01_01_000001_create_cache_table.php`. Однако, если по какой-то причине она отсутствует, вы можете создать её самостоятельно с помощью команды `cache:table`.

```shell
php imhotep cache:table

php imhotep migrate
```

<a name="interaction"></a>
## Взаимодействие

<a name="store"></a>
### Доступ к кэш-хранилищу

Для получения доступа к определенному кэш-хранилищу из списка, указанного в массиве `stores` конфигурационного файла `config/cache.php`, можно использовать один из следующих методов:

```php
// Используя фасад Cache
Cache::store('array')->set('foo', 'bar');

// Используя вспомогательную функцию
cache('file')->set('foo', 'bar');
```

<a name="has"></a>
### Проверка наличия данных

Чтобы узнать, присутствует ли ключ в кэше, используйте метод `has`. Если же вы хотите убедиться, что ключа нет в кэше, примените метод `missing`:

```php
// Проверяем наличие ключа
if (Cache::has('key')) {
    // ...
}

// Проверка на отсутствие ключа
if (Cache::missing('key')) {
    // ...
}
```

<a name="get"></a>
### Получение данных

Используйте метод `get`, чтобы получить значение по его ключу. Если ключ не найден, метод вернёт значение `null`. Вторым аргументом можно указать значение по умолчанию или передать замыкание, которое будет возвращать это значение в случае, если ключ не будет найден.

```php
// Вернет значение ключа или null
$value = Cache::get('key');
$value = cache()->get('key');

// Если значения не существует, будет возвращено 'default'
$value = Cache::get('key', 'default');
$value = cache()->get('key', 'default');

// Вариант с замыканием
$value = Cache::get('key', function () {
    return 'default';
});
```

Метод `many` возвращает массив значений переданных ключей. Для отсутствующих значений ключей будет возвращено `null`.

```php
$values = Cache::many('key1', 'key2', 'key3');
```

<a name="set"></a>
### Сохранение данных

Для сохранения данных в кэше воспользуйтесь методом `put`. Третьим аргументом, который является необязательным, можно указать время жизни данных в кэше. По истечении этого времени данные будут удалены. Если этот аргумент опущен, то он имеет значение 0, что означает, что данные будут храниться в кэше вечно.

```php
// Сохранить элемент на время указанное в конфигурации
Cache::put('key', 'value', 0);

// Сохранить элемент в кэше вечно
Cache::put('key', 'value', 0);

// Альтернативный вариант вечного хранения
Cache::forever('key', 'value');

// Сохранить элемент на 5 минут
Cache::put('key', 'value', ttl: 60 * 5);
```

Иногда возникает необходимость сохранить данные только в том случае, если они отсутствуют в кэше. Для этого можно воспользоваться методом `add`. Этот метод вернёт `true`, если данные были успешно сохранены, в противном случае `false`.

```php
// Добавить если отсутствует и
// хранить в течение времени указанного в конфигурации
Cache::add('key', 'value');

// Добавить если отсутствует и хранить вечно
Cache::add('key', 'value');

// Добавить если отсутствует и хранить 5 минут
Cache::add('key', 'value', ttl: 300);
```

<a name="set-many"></a>
#### Сохранение массива элементов

Для сохранения массива элементов, используйте метод `putMany`:

```php
// Хранить элементы на время указанное в конфигурации
Cache::putMany([
    ['key1' => 'value1'],
    ['key2' => 'value2'],
]);

// Хранить элементы в кэше вечно
Cache::putMany([
    ['key1' => 'value1'],
    ['key2' => 'value2'],
], 0);

// Сохранить элементы на 5 минут
Cache::putMany([
    ['key1' => 'value1'],
    ['key2' => 'value2'],
], ttl: 300);
```

<a name="increment-decrement"></a>
#### Increment / Decrement

Для увеличения или уменьшения целых чисел в кэше можно воспользоваться методами `increment` и `decrement`. В качестве второго необязательного аргумента они принимают величину, на которую нужно увеличить или уменьшить значение.

Эти методы возвращают обновленное значение или `false` в случае возникновения ошибки при сохранении. Если в кэше нет соответствующего значения, он будет добавлен автоматически.

```php
$value = Cache::increment('key'); // return 1
$value = Cache::increment('key', 10); // return 11
$value = Cache::decrement('key', 5); // return 6
```

<a name="remember"></a>
#### Извлечь и сохранить

Иногда необходимо извлечь значение элемента из кэша и в случае его отсутствия сохранить значение по умолчанию. Для этого в вашем распоряжении методы `remember` и `rememberForever`:

```php
// Данные хранятся в течение времени, указанного в конфигурации.
$value = Cache::remember('key', function () {
    return [...];
});

// Данные хранятся в течение 5 минут
$value = Cache::remember('key', function () {
    return [...];
}, ttl: 300);

// Данные хранятся вечно
$value = Cache::rememberForever('key', function () {
    return [...];
});
```

<a name="delete"></a>
### Удаление данных

Чтобы удалить элемент по ключу, воспользуйтесь методом `delete`. В случаях, когда требуется очистить весь кэш, примените метод `flush`.

```php
// Удалить значение по ключу
Cache::delete('key');

// Очистить весь кэш
Cache::flush();
```

<a name="create-driver"></a>
## Создание своего драйвера

Для создания собственного драйвера кэша, создайте класс, который реализует интерфейс `Imhotep\Contracts\Cache\Store`, например:

```php
<?php

namespace App\Extensions;

use Imhotep\Contracts\Cache\Store;

class MyCacheStore implements Store
{
    // ...
}
```

После создания класса и реализации всех его методов можно завершить процесс регистрации драйвера, вызвав метод `Cache::extend` в методе `register` класса `App\Providers\AppServiceProvider`:

```php
<?php

namespace App\Providers;

use Imhotep\Framework\Providers\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->booting(function () {
            Cache::extend('mycache', function () {
                return Cache::repository(new MyCacheStore());
            });
        });
    }
    
    // ..
}
```

Обратите внимание, что для вызова метода `Cache::extend` необходимо использовать обработчик `booting`. Это гарантирует, что ваш драйвер будет зарегистрирован после регистрации базовых классов системы кэширования.