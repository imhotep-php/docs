<a name="configuration"></a>
# Конфигурирование

<!-- NavigationBegin -->
- [Введение](#introduction)
- [Среда окружения](#environment)
  - [Дополнительные файлы окружения](#additional-files-env)
  - [Типы данных](#types-env)
  - [Получение значений окружения](#getting-values-env)
  - [Определение текущего окружения](#current-env)
  - [Безопасность данных](#security-env)
- [Получение значений конфигурации](#getting-values-config)
- [Изменение значений конфигурации](#changing-values-config)
- [Кеширование конфигурации](#caching-config)
- [Режим отладки](#debug-mode)
<!-- NavigationEnd -->

<a name="introduction"></a>
## Введение

Конфигурация — это основа, на которой строится ваше приложение. Она позволяет настроить подключение к базам данных, систему кеширования, почтовый сервер, язык сайта, часовой пояс и множество других параметров.

Все файлы конфигурации находятся в папке `/config`. Каждый параметр имеет комментарий, который описывает его назначение и доступные варианты значений. Изучите эти файлы и освойте их содержимое, чтобы лучше понимать, как работает ваше приложение.

<a name="environment"></a>
## Среда окружения

Очень часто возникает необходимость иметь разные конфигурации в зависимости от среды, в которой работает приложение. Например, на тестовом сервере можно использовать одни параметры доступа к базе данных, а на продакшн-сервере — другие.

Для решения этой задачи в корне вашего приложения предусмотрен файл `/.env`, который содержит основные значения конфигурации. Эти значения считываются файлами конфигурации, расположенными в папке `/config`, с помощью функции `env()`.

> Важно отметить, что любая переменная из файла /.env может быть переопределена внешней переменной из окружения на уровне сервера или системы.

<a name="additional-files-env"></a>
### Дополнительные файлы окружения

Перед загрузкой переменных окружения приложение проверяет, была ли задана переменная среды `APP_ENV` извне или указан аргумент CLI `--env`. В этом случае Imhotep загрузит файл `/.env.[APP_ENV]`, а если его не существует, то основной файл `/.env`.

<a name="types-env"></a>
### Типы данных

Все переменные окружения анализируются как строки, поэтому были разработаны специальные зарезервированные значения, позволяющие возвращать широкий диапазон типов из функции `env()`:

| Значение в `.env` | Значение из `env()` |
|-------------------|---------------------|
| true              | (bool) true         |
| (true)            | (bool) true         |
| false             | (bool) false        |
| (false)           | (bool) false        |
| empty             | (string) ''         |
| (empty)           | (string) ''         |
| (null)            | (null) null         |
| (null)            | (null) null         |

Если необходимо определить переменную со значением, содержащим пробел, его следует заключить в двойные кавычки:

```php
APP_NAME = "My Application"
```

<a name="getting-values-env"></a>
### Получение значений окружения

Чтобы получить значение переменной окружения, вы можете воспользоваться суперглобальной переменной `$__ENV` или глобальной функцией `env()`. Пример из файла `/config/app.php`:

```php
'debug' => env('APP_DEBUG', false)
```

Вторым аргументом в функцию `env()` можно передать значение по умолчанию, которое будет возвращено, если указанный ключ отсутствует в среде окружения.

<a name="current-env"></a>
### Определение текущего окружения

Текущее окружение определяется из переменной `APP_ENV`.

<a name="security-env"></a>
### Безопасность данных

Важно исключить файл `/.env` из системы контроля версий вашего приложения, так как он содержит конфиденциальную информацию о доступах к базе данных, облаку S3 и другим ресурсам. Если эта информация попадет к злоумышленникам, это может создать серьезную угрозу безопасности данных вашего приложения.


<a name="getting-values-config"></a>
## Получение значений конфигурации

Для получения значений параметров конфигурации можно воспользоваться фасадом `Imhotep\Facades\Config` или глобальной функцией `config()`. Эти функции доступны из любого места вашего приложения:

```php
use Imhotep\Facades\Config;

$value = Config::get('app.name');
$value = config('app.name');

// Вторым параметром задается значение по умолчанию,
// оно будет возвращено если значение в конфигурации отсутствует
$value = Config::get('app.timezone', 'Europe/Moscow');
$value = config('app.timezone', 'Europe/Moscow');

// Вторым значением допустимо передать замыкание, которое вернет значение по умолчанию
$value = config('app.language', function () {
    return 'ru';
});
$value = config('app.language', fn () => 'ru');
```

Для облегчения восприятия и статического анализа кода доступны типизированные методы получения значения. Если полученное значение не соответствует ожидаемому типу, будет выброшено исключение:

```php
use Imhotep\Facades\Config;

$value = Config::string('config-key');
$value = config()->string('config-key');

$value = Config::int('config-key');
$value = config()->int('config-key');

$value = Config::float('config-key');
$value = config()->float('config-key');

$value = Config::bool('config-key');
$value = config()->bool('config-key');

$value = Config::array('config-key');
$value = config()->array('config-key');
```


<a name="changing-values-config"></a>
## Изменение значений конфигурации

Чтобы изменить значения параметров конфигурации во время работы приложения, используйте следующие варианты:

```php
use Imhotep\Facades\Config;

Config::set('app.language', 'ru');
config()->set('app.language', 'ru');

// Если необходимо изменить несколько значений
Config::set(['app.language' => 'ru', 'app.locale' => 'ru']);
config()->set(['app.language' => 'ru', 'app.locale' => 'ru']);
```

<a name="caching-config"></a>
## Кеширование конфигурации

Для ускорения загрузки приложения на продакшн-сервере рекомендуется объединить все файлы конфигурации в один файл с помощью консольной команды:

```shell
php imhotep config:cache
```

> **Важно!** Кеширование конфигурации следует выполнять только на продакшн сервере, когда значения параметров больше не требуют изменений.

После кеширования конфигурации файл среды окружения `/.env` больше не будет загружаться приложением. Поэтому функция `env()` будет возвращать переменные окружения, которые были переданы системой. **Убедитесь, что функция `env()` вызывается только из файлов конфигурации.**

Чтобы удалить кеш конфигурации, используйте команду:

```shell
php imhotep config:clear
```

<a name="debug-mode"></a>
## Режим отладки

Во время разработки приложения важно получать больше информации об ошибках. Для этого в файле `app/config.php` значение параметра
`debug` должно быть установлено в `true`. По умолчанию значение этого параметра настраивается через переменную среды окружения `APP_DEBUG` из файла `.env`.

> **Важно!** Когда приложение будет выложено в продакшн, значение параметра `debug` должно быть `false`. Если оставить его равным `true`, существует риск раскрытия конфиденциальных данных пользователям приложения.