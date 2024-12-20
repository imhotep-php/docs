# Структура каталогов

## Введение

Структура приложения Imhotep предоставляет отличную основу как для крупных, так и для небольших проектов. Однако вы можете адаптировать её под свои потребности.

Imhotep не ограничивает расположение классов, при условии, что Composer может автоматически их загружать.

## Корневой каталог

### Каталог app
Каталог `/app` служит хранилищем основного кода вашего приложения. В ближайшее время мы подробно рассмотрим его структуру, но уже сейчас можно сказать, что почти все классы вашего приложения будут находиться именно здесь.

### Каталог bootstrap
В каталоге `/bootstrap` располагается файл `app.php`, который отвечает за загрузку фреймворка. Здесь же находится каталог `cache`, где хранятся файлы, сгенерированные фреймворком для оптимизации производительности, такие как файлы кеша конфигурации, маршрутов и других служб.

### Каталог config
Как следует из названия, каталог `/config` содержит все файлы конфигурации вашего приложения. Рекомендуем внимательно изучить все эти файлы, чтобы ознакомиться со всеми доступными параметрами.

### Каталог database
Каталог `/database` служит хранилищем для миграций ваших баз данных. При желании вы также можете использовать этот каталог для хранения SQLite БД.

### Каталог public
Каталог `/public` включает файл `index.php`, который служит точкой входа для всех запросов, поступающих в ваше приложение, а также настраивает автозагрузку. В этом каталоге также располагаются ваши ресурсы, такие как изображения, JavaScript и CSS.

### Каталог resources
Каталог `/resources` служит хранилищем ваших шаблонов и необработанных, нескомпилированных ресурсов, включая JavaScript и CSS.

### Каталог routes
В каталоге `/routes` находятся все маршруты вашего приложения. По умолчания в Imhotep включены два файла маршрутов: `web.php` и `console.php`.

В файле `web.php` перечислены маршруты, для обработки которых Imhotep применяет группу middleware `web`, настраиваемую в файле `/app/Http/Kernel.php`. Благодаря этому обеспечивается состояние сессии, защита от атак CSRF и шифрование файлов cookie.

В файле `console.php` вы можете определить все свои анонимные консольные команды. Каждое замыкание будет связано с соответствующим экземпляром команды, что позволяет легко взаимодействовать с методами ввода и вывода каждой команды. Хотя этот файл не отвечает за обработку HTTP-запросов, он устанавливает точки входа (маршруты) в ваше консольное приложение.

### Каталог storage
Каталог `/storage` служит хранилищем для различных файлов, включая логи, скомпилированные шаблоны, файлы сессий, кеш и другие данные. Этот каталог подразделяется на три подкатегории: `app`, `framework`, и `logs`. 

В каталоге `/storage/app` хранятся файлы, созданные вашим приложением.

Каталог `/storage/framework` содержит файлы, сгенерированные фреймворком.

Каталог `/storage/logs` служит для хранения логов вашего приложения.

### Каталог vendor
Каталог `/vendor` содержит ваши Composer-зависимости.