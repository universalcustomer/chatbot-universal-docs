.. _configuration:

=============
Configuration
=============

General suggestions
===================

* I am using  Ubuntu 19.10
* Server version: Apache/2.4.41
* mysql  Ver 8.0.20
* php 7.4.8
* VPS with 40 Gb and 2 Gb ram

Apache
======

* in case of the server put the project in ``/var/www/html``

* change permissions of the folder */storage* and all it files and directories inside of the project:

    .. code:: bash

        chmod -R 774 storage

* change owner and group of the folder */storage* and all it files and directories inside of the project to *www-data*:

    .. code:: bash

        chown -R www-data:www-data storage

* in your virtual host (in my case ``/etc/apache2/sites-available/chatbot-universal.conf``):
    .. code:: bash

        <VirtualHost *:80>
        ServerAdmin webmaster@localhost
        ServerName 00.000.000.000 # put your ip here
        ServerAlias chatbot-universal.com # put your domain here
        DocumentRoot /var/www/html/chatbot-universal/public # pointing to your public folder

        <Directory /var/www/html/chatbot-universal>
            AllowOverride All
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        RewriteEngine on
        RewriteCond %{SERVER_NAME} =chatbot-universal.com # put your server name here
        RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
        </VirtualHost>

* restart apache:
    .. code:: bash

        sudo service apache2 restart

Redis
=====

* you can change the port in ``/etc/redis/redis.conf``, default is 6379.
* run redis server:
    .. code:: bash

        sudo systemctl start redis

Laravel
=======

create ``.env`` file:
---------------------

.. code:: bash

    APP_NAME=
    APP_ENV=local
    APP_KEY=
    APP_DEBUG=true
    APP_URL="http://localhost:8000"

    LOG_CHANNEL=stack

    DB_CONNECTION=mysql
    DB_HOST=127.0.0.1
    DB_PORT=3306
    DB_DATABASE=oac
    DB_USERNAME=root
    DB_PASSWORD=

    BROADCAST_DRIVER=redis
    CACHE_DRIVER=file
    QUEUE_CONNECTION=redis
    SESSION_DRIVER=file
    SESSION_LIFETIME=120

    REDIS_HOST=127.0.0.1
    REDIS_PASSWORD=null
    REDIS_PORT=6379

    MAIL_MAILER=smtp
    MAIL_HOST=smtp.mailtrap.io
    MAIL_PORT=2525
    MAIL_USERNAME=null
    MAIL_PASSWORD=null
    MAIL_ENCRYPTION=null
    MAIL_FROM_ADDRESS=null
    MAIL_FROM_NAME="${APP_NAME}"

    AWS_ACCESS_KEY_ID=
    AWS_SECRET_ACCESS_KEY=
    AWS_DEFAULT_REGION=us-east-1
    AWS_BUCKET=

    PUSHER_APP_ID=
    PUSHER_APP_KEY=
    PUSHER_APP_SECRET=
    PUSHER_APP_CLUSTER=mt1

    MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
    MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"

    JWT_SECRET=
    MIX_APP_URL="${APP_URL}"

    GOOGLE_APPLICATION_CREDENTIALS=/*.json
    DIALOGFLOW_PROJECT_ID=

    TELEGRAM_BOT_TOKEN=

* APP_NAME
    Put the name you would like to. (better between "")

* APP_KEY
    For the key run the artisan command:

    .. code:: bash

        php artisan key:generate

* APP_URL
    you can put your url for the app, that one I use locally with ``php artisan serve``. When you upload the project, please change it to the appropriate.

    .. warning::

        Each time you change it, you have to run ``npm run prod`` command to rebuild the js files.

* DB_DATABASE
    put the name of your created database

* DB_USERNAME
    put the username of your created database

* DB_PASSWORD
    put the password of the user of your created database

* REDIS_PORT
    you can change the port, that one is the default.

* JWT_SECRET
    To generate the secret key run this command:

    .. code:: bash

        php artisan jwt:secret

* GOOGLE_APPLICATION_CREDENTIALS
    A root to ``.json`` key file from google cloud service account. You can know how to do it here: https://cloud.google.com/iam/docs/creating-managing-service-account-keys. you will need all permissions for Dialogflow.

* DIALOGFLOW_PROJECT_ID
    It's the id(name) of the project in your dialogflow general configuration.

* TELEGRAM_BOT_TOKEN
    *telegram bot token* from your bot created before.


database and queues:
--------------------

* make a migration to your database previously created and create 2 default users(admin and first user). You can find them and change them in ``database/seeds/DatabaseSeeder.php``:

    .. code:: bash

        php artisan migrate --seed

* initiate a process for laravel `queues <https://laravel.com/docs/8.x/queues>`_:

    .. code:: bash

        nohup sudo php artisan queue:work &

possible problems:
------------------

* error 500:
    * after every significant change in project clean cache:

        .. code:: bash

            php artisan optimize:clear

    * run command of composer:

        .. code:: bash

            php artisan clear-compiled
            composer dump-autoload
            php artisan optimize:clear


Laravel Echo Server
===================

run the command:

    .. code:: bash

        laravel-echo-server init

The cli tool will help you setup a laravel-echo-server.json file in the root directory of your project. This file will be loaded by the server during start up. You may edit this file later on to manage the configuration of your server.

    .. code:: bash

        nohup sudo laravel-echo-server start &

Telegram
========

run the url directly in your browser:

https://api.telegram.org/bot + *telegram bot token* (TELEGRAM_BOT_TOKEN) + /setWebhook?url= + *your app url* (APP_URL) + /api/telegram/webhook

That will put the telegram webhook of the chatbot on the mentioned url.
