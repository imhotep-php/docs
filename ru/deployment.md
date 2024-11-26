# Развертывание

## Введение

В этом руководстве мы подробно рассмотрим основные шаги, которые необходимо выполнить для успешного развертывания приложения Imhotep в рабочей среде. Чтобы приложение работало наилучшим образом, вам предстоит пройти через несколько простых, но важных этапов.

## Требование к серверу

Для корректной работы фреймворка Imhotep требуется, чтобы на вашем веб-сервере был установлен PHP не ниже указанной минимальной версии и соответствующие расширения:

- PHP >= 8.2
- Расширение PHP Ctype
- Расширение PHP cURL
- Расширение PHP DOM
- Расширение PHP Fileinfo
- Расширение PHP Filter
- Расширение PHP Hash
- Расширение PHP Mbstring
- Расширение PHP OpenSSL
- Расширение PHP PCRE
- Расширение PHP PDO
- Расширение PHP Session
- Расширение PHP Tokenizer
- Расширение PHP XML



## Конфигурация сервера Nginx

Если вы размещаете свое приложение на сервере с Nginx, вам поможет следующий конфигурационный файл. Он послужит основой для настройки вашего веб-сервера.

Важно, чтобы ваш веб-сервер направлял все запросы к файлу `/public/index.php` вашего приложения. Никогда не помещайте файл `index.php` в корень приложения, так как это может привести к открытию конфиденциальных файлов конфигурации приложения для посетителей из интернета.

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com;
    root /var/www/example.com/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_hide_header X-Powered-By;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```