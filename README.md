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

С функциональной точки зрения проект является реализацией
вступительного задания для программиста на Haskell в
сингапурскую компанию Zalora, описание которого
приведено [здесь](https://github.com/ownclo/shoesite)
и скопировано ниже. Вкратце, приложение являет собой
REST-сервис, принимаюший описания обуви в виде JSON
с картинкой в base64, сохраняющий метаданные в Postgres,
а картинку в виде файла в файловую систему, способный
показывать список товаров и позволяющий удалить товар
из базы.

Описание задания
----------------

Write a FastCGI or HTTP server in Haskell that provides a restful API for managing an inventory of shoes:

POST new shoes as a JSON body, with attributes "description", "color", "size", and "photo".
The "photo" attribute should be a base-64 encoded string representing a JPEG image (think "data URI").
For example,

    {"description": "SADIE Faux Suede Heels with Bow",
     "color": "red",
     "size": "35",
     "photo": "/9j/4AAQSkZJRgABAQAAAQABAAD/2wBDAAMCAgMCAgMDAwMEAwMEBQgFBQQEBQoHBwYIDAoMDAsKCwsNDhIQDQ4RDgsLEBYQERMUFRUVDA8XGBYUGBIUFRT/2wBDAQMEBAUEBQkFBQkUDQsNFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBT/wAARCAA2ACUDASIAAhEBAxEB/8QAGwABAQACAwEAAAAAAAAAAAAAAAkGBwEFCAr/xAAxEAABAwMDAgUCBAcAAAAAAAABAgMEBQYRAAchCBIJEyIxUTJhF0GCkSMzQlJxgaH/xAAaAQABBQEAAAAAAAAAAAAAAAAAAgMEBQcG/8QAKxEAAQMDAgMHBQAAAAAAAAAAAQACAwQFERIhIjFxBhMyQVFhgaHB0eHx/9oADAMBAAIRAxEAPwCqClJQMqISPknXPGpgeKRvTNuC7U7a0+oPRaVTGG35jbKyA9KWAtPdj3CElGPglWs18N/ruO4UWHtTuJPIuyKnyaRVJCuai2kfyXFH3eSBwT9YGD6h6mhICcLpKqxVFLRw1bjnvBqx6D+YPQ+xVCuPjTj40xpjTq5tONNMaaEKSviRbJ1qxt3pd5+WqTb1xL8xmTyfKfCfWyo/keO5Pyk8fSceHno1RolYj1ykyHYEyM+mTGlMEpU08ghQUlQ9iDj/AJq3XiFW8mv9LtwKUgKXCmQZCCRkpJkttEj9LqtS3bsuVZN6XFt7c7PlS4z6h5CvoWsDlSPstHaoEe4xqvk4HEBbXaqxl5tbYp/HHt8AAA/UZ/aq10T9TUfqe2Zh1qQW2rpppEGuREcdr4HDoH5IcHqH5A9w/p1v/wDfUH+nPqCqnRrvy7Ukoen27KIi1enoIzJjE5S4gHjzEZ7h+pOQFHVtLD3YtHcyzIV123X4VSoMsJDctDwSErUQA2sHBQvJAKFYIJxjUuN+oLL7pQOo5yGjhPL8LLP300wTpp1Ui6K+bHo249qVG27hhmfR56AiRHDq2ioBQUMLQQpJCkgggg8anB4ou1kml3dQL8hxVR1ONCDKlMjAUpHLKyf7u3KT9ko1T3H21je4W3dB3StSbblyQG6jSpiO1xpfuPggjkEHkEcjSHsDhhXNpuLrZVsn5tGxHqDz+x6gL5/Kg63e1WpzNecDDaXAl2ayj+IGyecge/8Aof4B9tbAvna/8MaYmo27PcqNjTXEOImJX6mncHsRISDjux3dqx6VDPbg9yR7wvjwmLSqSlu2zdtWpKlZwxMSiQ2PsOEq/cnWvZvhLXfPhsQHNzWDT2HFOtsrgLUkKPurt80DP31BMMgIwdlpMfaW2RytqYTpd5gt8uozg/K9PeH1uBeO4uxDU+7JaaoyxKMal1IvIcekR0oTkOkKKu9CypHrCVYSD6shRa7/AKP+lxXSxZVWoi7nkXM7UpglrUqMIzTJCAnCG+9fJxyrPOE8DHLU9uQBlZhc5Yp6yWWHGlxyMDA39lvzA0wNNNKVYnGuONNNCEOmmmhC/9k="
    }

GET a shoe as an HTML page listing the shoe details, where the photo is served as an <img> tag
with "src" pointing to a path on the local filesystem (i.e. the photo must be accessible as a local file, not as a data URI).
GET a list of shoes as an HTML page with hyperlinks to all available shoes.

YESOD
=====

Yesod отличается от других web-фреймворков на Haskell (Snap) так же,
как RoR отличается от Sinatra: первые предлагают сквозное решение,
берущее на себя и роутинг, и работу с БД, и темплейтинг, в
каком-то смысле навязывая свои стандартизованные (good practices)
решения в каждой из этих частей. Это может быть как большим преимуществом,
так и серьёзным недостатком фреймворка. В рамках данного проекта
было решено прибегнуть именно к такому фреймворку, потому что
он умеет управлять схемой и миграцией БД и позволяет быстро
перейти к прототипизированию бизнес-логики.

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

В ходе выполнения команды `yesod init` вам будет предложено
ответить на несколько вопросов. В результате появится папка
с указанным Вами именем, содержащая готовую инфраструктуру
yesod-проекта.

Routing
-------
Роутинг работает через `config/routes`:

	/ HomeR GET POST
	/echo/#Text EchoR GET
	/mirror MirrorR GET POST
	/blog BlogR GET POST
	/blog/#ArticleId ArticleR GET

	/shoes ShoesR GET POST
	/shoes/#ShoeId ShoeR GET DELETE
	/shoes/delete/#ShoeId DeleteShoeR POST

Вопросы защиты от эксплойта и выделения ID берёт на
себя Yesod.

Database
--------
Yesod самостоятельно генерирует схему БД и типы приложения
на основе описания в `config/models` (как ORM, только не O):

	User
		ident Text
		password Text Maybe
		UniqueUser ident
		deriving Typeable
	Email
		email Text
		user UserId Maybe
		verkey Text Maybe
		UniqueEmail email
	Shoe
		description Text
		color Text
		size Text
		UniqueShoe description color size

В этом примере указаны типы и поля для каждой сущности. Есть возможность
указывать ограничения типа FK, Unique и тп. Фреймворк генерирует схему
БД и типы для этих сущностей, которые можно использовать для простого
CRUD-взаимодействия с БД (и one-to-many запросов).

Параметры самой БД описываются в `config/postgresql.yml`:

	Default: &defaults
	  user: shoesite
	  password: shoesite
	  host: localhost
	  port: 5432
	  database: shoesite
	  poolsize: 10

Используется многопоточный пул подключений к БД, из которого по необходимости
выделяются подключения.

Templating
----------
Шаблонизация и генерация HTML, JavaScript и CSS осуществляется с помощью
файлов `.hamlet`, `.julius` и `.cassius` соответственно. Очень похоже на
шаблоны в Django и Rails. Шаблоны HTML умеют в итерацию, например `templates/shoes.hamlet`:

	<h1> List of shoes
	$if null shoes
		<p> There are no shoes here.
	$else
		<ul>
		$forall Entity shoeId shoe <- shoes
			<li>
				<a href=@{ShoeR shoeId} method=POST> #{shoeDescription shoe}

Шаблоны не очень приятно отлаживать, поэтому не стоит злоупотреблять
возможностью встраивать сложный код на Haskell в шаблон. Преимущественно
это используется для доступа к полям записи, например `templates/shoe.hamlet`:

	<dl class="gallery">
	  <dt>Description
	  <dd>#{shoeDescription shoe}
	  <dt>Color
	  <dd>#{shoeColor shoe}
	  <dt>Size
	  <dd>#{shoeSize shoe}
	  <dt>Photo
	  <dd>
		<img src=#{".." </> shoePhotoPath shoeId}>

	<form method=post action=@{DeleteShoeR shoeId}>
	  <input type=submit value="Delete">

Sample handler
--------------
Пример простого хэндлера, который отвечает на запрос `/shoe/#ShoeId GET` (см. Роутинг),
представлен ниже (`Handler/Shoe.hs`):

	getShoeR :: ShoeId -> Handler Html
	getShoeR shoeId = do
		shoe <- runDB $ get404 shoeId
		defaultLayout $ do
			setTitle . toHtml $ shoeDescription shoe
			$(widgetFile "shoe")

Следует отметить несколько моментов:

- `ShoeId` -- типизированный идентификатор для элемента. Это отражает один из ключевых инструментов yesod-а в частности и Haskell-я в целом: представление сущностей приложения на уровне типов. Тип `ShoeId` присутствует и в описании роутинга, и в описании структуры БД. Так, если перепутать и запросить у БД какой-то другой ID, то программа не скомпилируется.
- `get404` самостоятельно на основе типа поданного ему на вход параметра определяет, к какой сущности БД необходимо обратиться и на основе этого формирует соответствующий запрос. Сочетание лаконичности (в JAVA обычно пишут CRUD на каждую сущность и называют это DAO Pattern) и типобезопасности (можно генерировать запросы динамически, но для этого в языках с более слабой системой типов приходится жертвовать типобезопасностью) делает Yesod ценным инструментом веб-разработки.
- `$(widgetFile "shoe")` -- это вызов шаблонной функции, которая раскрывается в рендеринг HTML-кода на основе файлов `templates/shoe.{hamlet,julius,lucius}`.

New Handlers
------------
Новые методы можно добавлять вручную. Но для удобства разработчика,
как и в Ruby on Rails, добавлена возможность делать это автоматически
с помощью команды `yesod add-handler`:

	(master)➤ stack exec -- yesod add-handler
	Name of route (without trailing R): Cash
	Enter route pattern (ex: /entry/#EntryId): /cash/#CashId
	Enter space-separated list of methods (ex: GET POST): GET POST DELETE
	(master)➤ git status -sb
    ## master...origin/master
	M Application.hs
	M blah.cabal
	M config/routes
	?? Handler/Cash.hs
	?? test/Handler/CashSpec.hs
	(master)➤ git add --all
	(master)➤ git diff --cached
	diff --git a/Application.hs b/Application.hs
	index 6c84ccf..6908758 100644
	--- a/Application.hs
	+++ b/Application.hs
	@@ -34,6 +34,7 @@ import Handler.Common
	 import Handler.Home
	 import Handler.Shoes
	 import Handler.Shoe
	+import Handler.Cash
	 
	 -- This line actually creates our YesodDispatch instance. It is the second half
	 -- of the call to mkYesodData which occurs in Foundation.hs. Please see the
	diff --git a/Handler/Cash.hs b/Handler/Cash.hs
	new file mode 100644
	index 0000000..177f844
	--- /dev/null
	+++ b/Handler/Cash.hs
	@@ -0,0 +1,12 @@
	+module Handler.Cash where
	+
	+import Import
	+
	+getCashR :: CashId -> Handler Html
	+getCashR cashId = error "Not yet implemented: getCashR"
	+
	+postCashR :: CashId -> Handler Html
	+postCashR cashId = error "Not yet implemented: postCashR"
	+
	+deleteCashR :: CashId -> Handler Html
	+deleteCashR cashId = error "Not yet implemented: deleteCashR"
	diff --git a/blah.cabal b/blah.cabal
	index 3c3d934..4079514 100644
	--- a/blah.cabal
	+++ b/blah.cabal
	@@ -25,6 +25,7 @@ library
						  Handler.Home
						  Handler.Shoes
						  Handler.Shoe
	+                     Handler.Cash
	 
		 if flag(dev) || flag(library-only)
			 cpp-options:   -DDEVELOPMENT
	diff --git a/config/routes b/config/routes
	index df75e98..9069fb0 100644
	--- a/config/routes
	+++ b/config/routes
	@@ -8,3 +8,4 @@
	 /shoes ShoesR GET POST
	 /shoes/#ShoeId ShoeR GET DELETE
	 /shoes/delete/#ShoeId DeleteShoeR POST
	+/cash/#CashId CashR GET POST DELETE
	diff --git a/test/Handler/CashSpec.hs b/test/Handler/CashSpec.hs
	new file mode 100644
	index 0000000..232cb81
	--- /dev/null
	+++ b/test/Handler/CashSpec.hs
	@@ -0,0 +1,18 @@
	+module Handler.CashSpec (spec) where
	+
	+import TestImport
	+
	+spec :: Spec
	+spec = withApp $ do
	+
	+    describe "getCashR" $ do
	+        error "Spec not implemented: getCashR"
	+
	+
	+    describe "postCashR" $ do
	+        error "Spec not implemented: postCashR"
	+
	+
	+    describe "deleteCashR" $ do
	+        error "Spec not implemented: deleteCashR"
	+

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

[На основе инфы отсюда](https://gist.github.com/mtigas/952344)

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