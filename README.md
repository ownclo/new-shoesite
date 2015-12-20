Формулировка задания, общее описание решения
============================================

Реализовать веб-приложение на базе операционной системы
ubuntu linux, предусмотреть следующий функционал:

- СУБД
- Бизнес-логика
- Динамическая генерация HTML-страниц
- URL-based routing
- HTTPS-трафик
- Клиентские и серверные сертификаты

Для реализации первых четырёх функций использован
веб-фреймворк Yesod на языке Haskell, база данных
Postgres. Yesod предоставляет следующий функционал:

- Поддержка множества СУБД (`Database.Persistent`)
- Статически типизированный роутинг по URL
- Динамический шаблонизатор HTML/JavaScript/CSS (Hamlet/Lucius/Julius)
- HTTPS

Yesod использует свой собственный веб-сервер (WARP),
который использует легковесные потоки Haskell для
обеспечения производительности веб-приложения.

WARP не поддерживает клиентские сертификаты, поэтому
для реализации этого функционала был использован NGINX-прокси.

YESOD
=====

Init
----

Для разработки использовалась система контроля сборки
и менеджер Haskell-пакетов `stack`. Для создания
шаблона веб-приложения выполните:

    $ stack install yesod-bin-1.4.11 cabal-install
    $ stack exec yesod init
    $ cd project-name
    $ stack init
    $ stack build yesod-bin-1.4.11

Database
--------

Routing
-------

Templating
----------

APPROOT
-------

APPROOT должен указывать на слушающий порт NGINX (см. далее):

    $ export APPROOT=https://localhost

HTTPS
-----

Yesod умеет отдавать контент как по HTTP, так и по HTTPS.
Для того, чтобы использовать HTTPS между NGINX и сайтом,
просто укажите соответствующий порт в `backend` и `proxy_pass`
с префиксом HTTPS в конфиге nginx (см. ниже).

Bootstrap
---------

    $ stack exec -- yesod devel

Client-server certificates
==========================

Certificates
------------

*NOTE:* Organization & Common names should be different for each crt file. Self-signed certificate error will be thrown otherwise

Here is how to use it:

    $ curl -k --key client.key --cert client.crt "https://localhost"

CA cert
-------

*NOTE:* Organization & Common Name: Some human identifier for this server CA.

    $ openssl genrsa -des3 -out ca.key 4096
    $ openssl req -new -x509 -days 365 -key ca.key -out ca.crt

Client certificate
------------------

* generate rsa key
* generate csr
* generate crt (certificate), sign in with CA cert
* generate p12 for browser
* import p12 to browser (chrome: Settings -> Advanced -> Manage Certificates -> User -> Import)

*NOTE:* Organization & Common Name: Person name (different from CA's)

    $ openssl genrsa -des3 -out client.key 4096
    $ openssl req -new -key client.key -out client.csr
    $ openssl x509 -req -days 365 -in client.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out client.crt
    $ openssl pkcs12 -export -clcerts -in client.crt -inkey client.key -out client.p12

Server certificate
------------------

*NOTE:* Organization & Common Name: server name (different from CA's and client's)

    $ openssl genrsa -des3 -out server.key 4096
    $ openssl req -new -key server.key -out server.csr
    $ openssl x509 -req -days 365 -in server.csr -CA ca.crt -CAkey ca.key -set_serial 02 -out server.crt

Check out whether your certificates are OK
------------------------------------------

    $ openssl verify -purpose sslclient -CAfile ca.crt client.crt 
    $ openssl verify -purpose sslserver -CAfile ca.crt server.crt 

NGINX
=====

Для настройки NGINX в роли HTTPS-прокси с клиентскими
и серверными сертификатами нужно создать файл с
настройками соответствующего сайта (`/etc/nginx/sites-available/shoesite.ru`):

    upstream backend_shoesite {
      server 127.0.0.1:3000;
    }

    server {
      listen 443;
      server_name www.shoesite.ru shoesite.ru;

      ssl on;
      ssl_certificate /home/ownclo/workspace/haskell/blah/cert/server.crt;
      ssl_certificate_key /home/ownclo/workspace/haskell/blah/cert/server.key;
      ssl_client_certificate /home/ownclo/workspace/haskell/blah/cert/ca.crt;
      ssl_verify_client on;

      access_log /home/ownclo/learnspace/labs/web/second/nginx_ssl_access.log;
      error_log /home/ownclo/learnspace/labs/web/second/nginx_ssl_error.log debug;

      location / {
        proxy_pass http://backend_shoesite;
        include /etc/nginx/proxy_params;
      }
    }

Здесь осуществляется настройка nginx следующим образом: при запросе по
адресу `shoesite.ru` перенаправлять на бэкэнд. Отвечать и на HTTP (порт 80),
и на HTTPS (порт 443). Для активации данного сайта слинкуйте конфигурационный
файл в папку `sites-enabled`:

    $ sudo ln -s /etc/nginx/sites-available/shoesite.ru /etc/nginx/sites-enabled/
    $ sudo process restart nginx

В этом же конфигурационном файле можно добавить и инструкции по отдаче
статического контента (`.jpg` etc.) и таким образом превратить nginx
в кэширующий прокси.