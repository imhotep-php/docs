# Шифрование

- [Введение](#introduction)
- [Конфигурирование](#configuration)
- [Шифрование значения](#encrypt)
- [Расшифровка значения](#decrypt)

<a name="introduction"></a>
## Введение

Сервис шифрования Imhotep предоставляет простой и удобный интерфейс для шифрования и дешифрования текста через OpenSSL. Все зашифрованные значения подписываются с использованием кода аутентификации сообщения (MAC), поэтому их базовое значение не может быть изменено или подделано после шифрования.

<a name="configuration"></a>
## Конфигурирование

Для использования шифровальщика необходимо настроить параметр `key` в конфигурационном файле `config/app.php`. Это значение конфигурации управляется переменной окружения `APP_KEY` в файле `.env`.

Используйте команду `php imhotep key:generate` для создания нового ключа безопасности.

> По умолчанию, ключ генерируется автоматически во время установки Imhotep.

<a name="encrypt"></a>
## Шифрование значения

Для шифрования значения, используйте метод `encryptString` фасада `Crypt` или глобальный помощник `encryptString`. Если значение не может быть правильно зашифровано, будет выброшено исключение `Imhotep\Contracts\Encryption\EncryptException`:

```php
use Imhotep\Contracts\Encryption\EncryptException;

try {
    $encryptedValue = Crypt::encryptString("Secret text");
    // или
    $encryptedValue = encryptString("Secret text");
} catch (EncryptException $e) {
    // ...
}
```

<a name="decrypt"></a>
## Расшифровка значения

Для расшифровки значения, используйте метод `decryptString` фасада `Crypt` или глобальный помощник `decryptString`. Если значение не может быть правильно расшифровано, будет выброшено исключение `Imhotep\Contracts\Encryption\DecryptException`:

```php
use Imhotep\Contracts\Encryption\DecryptException;

try {
    $decryptedValue = Crypt::decryptString($encryptedValue);
    // или
    $decryptedValue = decryptString($encryptedValue);
} catch (DecryptException $e) {
    // ...
}
```