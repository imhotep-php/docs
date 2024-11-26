# Базы данных: Введение

## Введение

Imhotep позволяет взаимодействовать с популярными базами данными через простой интерфейс. В настоящее время поддерживаются следующие базы данных: PostgreSQL, MySQL, MariaDB, SQLite.

### Конфигурирование

Все параметры соединения с базами данными находятся в конфигурационном файле `config/database.php`.

#### Подключение к SQLite

База данных SQLite хранится в одном файле в локальной файловой системе. Для создания базы, используйте команду `touch` в консоли: `touch database/database.sqlite`. После создания базы, настройте переменные окружения в файле `.env`, чтобы они указывали абсолютный путь к базе в переменной `DB_DATABASE`:

```php
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database.sqlite
```

### Read & Write соединения


## Выполнение SQL-запросов

Для выполнения SQL-запросов используйте фасад DB. Данный фасад содержит методы для каждого типа запроса select, insert, update, delete и statement, где первым аргументом принимают SQL-запрос, а вторым связанные параметры, необходимые для выполнения запроса (например: значения выражений WHERE).

Используйте связанные параметры, вместо подстановки значений напрямую в SQL-запрос, это защитит приложение от SQL-инъекций.

### SELECT

Метод `select` всегда возвращает массив данных. Все элементы массива будут объектом класса `stdClass`, представляющие запись из базы данных.

```php
use Imhotep\Facades\DB;

$users = DB::select("SELECT * FROM users");

foreach ($users as $user) {
    echo $user->name;
}
```

Если запрос имеет условия, используйте связанные параметры для передачи их значений:

```php
// Обычное связывание, через символ "?"
$users = DB::select("SELECT * FROM users WHERE active = ?", [1]);

// Использование именованных привязок
$users = DB::select("SELECT * FROM users WHERE active = :active", ['active' => 1]);
```

### INSERT

Результатом выполнения INSERT запроса, будет количество вставленных записей.

```php
// Обычное связывание, через символ "?"
$affected = DB::insert("INSERT INTO users (id, name) VALUES (?, ?)", [1, 'John']);

// Использование именованных привязок
$affected = DB::insert("INSERT INTO users (id, name) VALUES (:id, :name)", ['id' => 1, 'name' => 'John']);
```

### UPDATE

Результатом выполнения UPDATE запроса, будет количество обновленных записей.

```php
// Обычное связывание, через символ "?"
$affected = DB::update("UPDATE users SET age = ? WHERE name = ?", [24, 'John']);

// Использование именованных привязок
$affected = DB::update("UPDATE users SET age = :age WHERE name = :name", ['age' => 24, 'name' => 'John']);
```

### DELETE

Результатом выполнения DELETE запроса, будет количество удаленных записей.

```php
// Обычное связывание, через символ "?"
$affected = DB::delete("DELETE FROM users WHERE name = ?", ['John']);

// Использование именованных привязок
$affected = DB::delete("DELETE FROM users WHERE name = :name", ['name' => 'John']);
```

### Другие запросы

Некоторые операторы SQL-запросов не возвращают значений. Для этих операций можно использовать метод `statement` фасада `DB`, который в случае успешного выполнения вернет `true`:

```php
$result = DB::statement("DROP TABLE users");
```

## Несколько соединений к БД

Если в конфигурационном файле `config/database.php` определено несколько соединений, для получения доступа к каждому из них, используйте метод `connection` фасада `DB`, передав в качестве первого аргумента имя соединения, указанному в конфигурационном файле.

```php
use Imhotep\Facades\DB;

$users = DB::connection('sqlite')->select(...);
```

## Прослушивание событий запроса

При необходимости, можно указать замыкание, которое будет вызываться для каждого SQL-запроса, используя метод `listen` фасада `DB`. Этот метод может быть полезен для логирования или отладки. Зарегистрировать замыкание удобней всего в методе `boot` в `AppServiceProvider`:

```php
<?php

namespace App\Providers;

use Imhotep\Facades\DB;
use Imhotep\Framework\Providers\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    // ...
    
    public function boot()
    {
        DB::listen(function ($query) {
            // $query->sql;
            // $query->bindings;
            // $query->time;
        });
    }
}
```

## Транзакционные запросы

Используйте метод `transaction` фасада `DB`, для выполнения транзакционных запросов. Если во время обработки транзакции произойдет ошибка, транзакция автоматически откатится и будет выброшено исключение. 

```php
use Imhotep\Facades\DB;

DB::transaction(function () {
    DB::update('UPDATE users SET votes = 1');

    DB::delete('DELETE FROM posts');
});
```

### Обработка блокировок

Метод `transaction` принимает необязательный второй аргумент, который определяет, сколько раз транзакция должна повториться при появлении блокировок. Как только эти попытки будут исчерпаны, будет выброшено исключение:

```php
use Imhotep\Facades\DB;

DB::transaction(function () {
    DB::update('UPDATE users SET votes = 1');

    DB::delete('DELETE FROM posts');
}, 5);
```

### Ручная обработка транзакций

Если необходимо иметь полный контроль над откатами и фиксацией, используйте метод `beginTransaction` фасада `DB`:

```php
use Imhotep\Facades\DB;

DB::beginTransaction();
```

Для отката транзакции, используйте методе `rollBack`:

```php
DB::rollBack();
```

И для фиксации транзакции, используйте метод `commit`:

```php
DB::commit();
```