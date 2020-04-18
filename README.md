## СБОРКА БАЗОВОГО САЙТА (ПРИЛОЖЕНИЯ) НА LARAVEL FRAMEWORK 7.X 

Данный материал содержит инструкцию по сборке базового сайта (приложения) на Laravel Framework 7.x под операционной системой Windows с установленным Open Server. Open Server Panel — это портативная серверная платформа и программная среда, созданная специально для веб-разработчиков с учётом их рекомендаций и пожеланий.
Скачать Open Server можно здесь https://ospanel.io/. Установка его абсолютно интуитивно понятна.
Весь материал, касающийся именно базовой настройки Laravel, так же справедлив и для таких операционных систем, как Mac OS и Linux, естественно с учётом их особенностей.

##	Установка Composer на OpenServer

1) переходим к текущему модулю PHP, например:

D:\OSPanel\modules\php\PHP_7.3-x64

2) запускаем:

php -r "readfile('https://getcomposer.org/installer');" | php

или Composer-Setup.exe, указывая  D:\OSPanel\modules\php\PHP_7.3-x64 

3) Проверяем какая версия стоит командой: 

php composer.phar -V

4) Создаём .bat файл и в дальнейшем можно будет работать без указания php composer.phar, просто указывая composer:

echo @php "%~dp0composer.phar" %*>composer.bat

##	Установка Laravel

1.	Запускаем консоль OpenServer
2.	Переходим в папку \OSPanel\domains
3.	Запускаем команду
composer create-project --prefer-dist laravel/laravel laravel.empty.blog
	laravel.empty.blog –  каталог нашего проекта
	
4.	Устанавливаем Node.JS https://nodejs.org/en/
5.	Запускаем OpenServer, и идем в настройки. Во вкладке «Сервер» в поле «Настройка использования переменной Path» выставляем «Свой Path + userdata/config/path.txt». Создаем файл в папке \OpenServer\userdata\config\ под именем path.txt следующего содержания:
	C:\Program Files\nodejs\
6.	Перезапускаем OpenServer
7.	Проверяем npm –i
8.	Запускаем консоль, переходим в паку проекта \OSPanel\domains\laravel.empty.blog
9.	Запускаем установку npm install
10.	В OpenServer идем в настройки. Во вкладке «Домены» добавляем папку \OSPanel\domains\laravel.empty.blog\public  -> имя домена laravel.empty.blog 
11.	Проверяем в браузере http://laravel.empty.blog   
12.	В OpenServer идём в «Дополнительно» phpmyadmin (логин root без пароля)
13.	Создаём базу laravel.empty.bl utf8mb4_general_ci
14.	Редактируем файл .env в \OSPanel\domains\laravel.empty.blog\

APP_NAME=Laravel.Empty.Blog
APP_URL=http://laravel.empty.blog
DB_DATABASE=laravel.empty.bl

Можно настроить smtp (на примере Яндекс):

MAIL_DRIVER=smtp
MAIL_HOST=smtp.yandex.ru
MAIL_PORT=587
MAIL_USERNAME=ВАШ_ЛОГИН
MAIL_PASSWORD=ВАШ_ПАРОЛЬ
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=ВАШ_EMAIL
MAIL_FROM_NAME="${APP_NAME}"

15. В консоли в папке \OSPanel\domains\laravel.empty.blog\ запускаем команду 
       php artisan config:cache

16. В :\OSPanel\domains\laravel.empty.blog\resources\views\welcome.blade.php 
меняем:
<title>Laravel</title>
на <title>{{ config('app.name', 'Laravel') }}</title>
Надпись Laravel в <div class="title m-b-md">
на {{ config('app.name', 'Laravel') }} ;) 


##	Запуск сайта (приложения) на Laravel.

17. Проверяем в браузере http://laravel.empty.blog

Мы должны увидеть: Laravel.Empty.Blog ;)    

18. Открываем папку нашего проекта в PHPStorm или в другой IDE

19. Версию установленного Laravel можно поверить командой (в консоли в папке проекта):

php artisan –V

В нашем случае это Laravel Framework 7.6.2

20. Список всех команд: php artisan

21. Модифицируем файл app/Providers/AppServiceProvider.php
Без этих изменений может возникать ошибка:
Syntax error or access violation: 1071 Specified key was too long; max key length is 767 bytes
Дело в том, что в Laravel 5.4 изменилась кодировка по умолчанию для базы данных (теперь это utf8mb4 для поддержки emoji). Ошибка Specified key was too long error проявляется только для MySQL ниже v5.7.7 и в старых версиях MariaDB.

Итак, в app/Providers/AppServiceProvider.php добавляем:

1)	Добавьте строку в блоке use: use Illuminate\Support\Facades\Schema;
2)	Добавьте в метод boot строку:  Schema::defaultStringLength(191); 

22. Запускаем миграции: php artisan migrate 

Вариант сообщений при успешном выполнении миграций:

Migration table created successfully. 
Migrating: 2014_10_12_000000_create_users_table 
Migrated:  2014_10_12_000000_create_users_table (0.44 seconds) 
Migrating: 2019_08_19_000000_create_failed_jobs_table 
Migrated:  2019_08_19_000000_create_failed_jobs_table (0.24 seconds)

В базе данных laravel.empty.bl создались таблицы failed_jobs и users

23. Добавляем механизм авторизации пользователей

Каркас фронтенда, обычно поставляемый с предыдущими версиями Laravel, был перенесен в отдельный пакет laravel/ui. Это позволяет создавать и версионировать пользовательские интерфейсы отдельно от основного фреймворка. В результате этого изменения в дефолтном каркасе фреймворка не будет Bootstrap и Vue. 
Также была вынесена из фреймворка команда make:auth.Чтобы восстановить традиционный каркас Vue/Bootstrap, имеющийся в предыдущих версиях Laravel, вы можете установить пакет laravel/ui и использовать artisan команду ui для установки фронтенд каркаса.

Устанавливаем пакет: composer require laravel/ui

Устанавливаем фронтенд: php artisan ui vue –auth

Загружаем пакеты фронтенда: npm install

Запускаем нашу сборку в режиме разработчика: npm run dev

В результате последней команды будут скомпелированы файлы стилей и скриптов 
/css/app.css
/js/app.js


24. Процесс регистрирации.

Если всё сделано правильно, то перейдя на наш сайт http://laravel.empty.blog мы увидим в верхнем углу 2 пункта меню Login и Register.

Производим регистрацию нового пользователя – вводим имя, e-mail, пароль и подтверждение пароля. После устпешной регистрации вы должны увидеть надпись You are logged in!

Это значит, что регистрация прошла успешно и Вы уже авторизованы на сайте! 

Поздравляем! Базовая сборка сайта на Laravel 7 произведена, и теперь Ваш сайт полностью готов к дальнейшему развитию!

25. Git

Весь код к данному материалу находиться в общедоступном репозитории
https://github.com/Best-ITPro/LaravelEmptyBlog_Blank.git 

Вы можете скачать и использовать его для создания Ваших приложений.

В дальнейшем будет опубликована более расширенная версия сайта (приложения), которая будет содержать механизм публикации статей в различных категорях.


С уважением,
Команда Best IT Pro
http://best-itpro.ru
 


## About Laravel

Laravel is a web application framework with expressive, elegant syntax. We believe development must be an enjoyable and creative experience to be truly fulfilling. Laravel takes the pain out of development by easing common tasks used in many web projects, such as:

- [Simple, fast routing engine](https://laravel.com/docs/routing).
- [Powerful dependency injection container](https://laravel.com/docs/container).
- Multiple back-ends for [session](https://laravel.com/docs/session) and [cache](https://laravel.com/docs/cache) storage.
- Expressive, intuitive [database ORM](https://laravel.com/docs/eloquent).
- Database agnostic [schema migrations](https://laravel.com/docs/migrations).
- [Robust background job processing](https://laravel.com/docs/queues).
- [Real-time event broadcasting](https://laravel.com/docs/broadcasting).

Laravel is accessible, powerful, and provides tools required for large, robust applications.

## Learning Laravel

Laravel has the most extensive and thorough [documentation](https://laravel.com/docs) and video tutorial library of all modern web application frameworks, making it a breeze to get started with the framework.

If you don't feel like reading, [Laracasts](https://laracasts.com) can help. Laracasts contains over 1500 video tutorials on a range of topics including Laravel, modern PHP, unit testing, and JavaScript. Boost your skills by digging into our comprehensive video library.

## Laravel Sponsors

We would like to extend our thanks to the following sponsors for funding Laravel development. If you are interested in becoming a sponsor, please visit the Laravel [Patreon page](https://patreon.com/taylorotwell).

- **[Vehikl](https://vehikl.com/)**
- **[Tighten Co.](https://tighten.co)**
- **[Kirschbaum Development Group](https://kirschbaumdevelopment.com)**
- **[64 Robots](https://64robots.com)**
- **[Cubet Techno Labs](https://cubettech.com)**
- **[Cyber-Duck](https://cyber-duck.co.uk)**
- **[British Software Development](https://www.britishsoftware.co)**
- **[Webdock, Fast VPS Hosting](https://www.webdock.io/en)**
- **[DevSquad](https://devsquad.com)**
- [UserInsights](https://userinsights.com)
- [Fragrantica](https://www.fragrantica.com)
- [SOFTonSOFA](https://softonsofa.com/)
- [User10](https://user10.com)
- [Soumettre.fr](https://soumettre.fr/)
- [CodeBrisk](https://codebrisk.com)
- [1Forge](https://1forge.com)
- [TECPRESSO](https://tecpresso.co.jp/)
- [Runtime Converter](http://runtimeconverter.com/)
- [WebL'Agence](https://weblagence.com/)
- [Invoice Ninja](https://www.invoiceninja.com)
- [iMi digital](https://www.imi-digital.de/)
- [Earthlink](https://www.earthlink.ro/)
- [Steadfast Collective](https://steadfastcollective.com/)
- [We Are The Robots Inc.](https://watr.mx/)
- [Understand.io](https://www.understand.io/)
- [Abdel Elrafa](https://abdelelrafa.com)
- [Hyper Host](https://hyper.host)
- [Appoly](https://www.appoly.co.uk)
- [OP.GG](https://op.gg)
- [云软科技](http://www.yunruan.ltd/)

## Contributing

Thank you for considering contributing to the Laravel framework! The contribution guide can be found in the [Laravel documentation](https://laravel.com/docs/contributions).

## Code of Conduct

In order to ensure that the Laravel community is welcoming to all, please review and abide by the [Code of Conduct](https://laravel.com/docs/contributions#code-of-conduct).

## Security Vulnerabilities

If you discover a security vulnerability within Laravel, please send an e-mail to Taylor Otwell via [taylor@laravel.com](mailto:taylor@laravel.com). All security vulnerabilities will be promptly addressed.

## License

The Laravel framework is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).
