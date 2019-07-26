# PHP image with Laravel

This repo is used to generate a base PHP image that includes Laravel, the well-known PHP framework
for web apps.  All the usual extensions are already built into the image, including support for mysql. `composer` is
also installed in the image to allow you to manage your packages within Docker.  Currently 
PHP version 7.3 is supported, but new versions will be added as they become stable.

## How to create a new Laravel microservice based on this image

Let's say you want to create a new service called `testlaravel` to work alongside our other microservices.

First create your top level dev folder which will be your repo in github:

```bash
mkdir ~/dev/testlaravel_service
cd ~/dev/testlaravel_service
```

Then run Laravel to create your new application skeleton into a subfolder.  Using a subfolder is best practice
to avoid copying unnecessary files/folders (like .git etc) into the final docker image.

```bash
docker run -it -v `pwd`:/var/www/html mumsnet/php-fpm-laravel:7.3 laravel new testlaravel
```

Configure Laravel:

Edit the `.env` file and set `APP_NAME=Testlaravel`, 
and `APP_URL=https://lhYOURNAME.devmn.net/service/testlaravel`

Then edit `routes/web.php` and wrap all the routes in a group like this:

```php
Route::prefix('service/testlaravel')->group(function () {
    // all existing routes eg:
    Route::get('/', function () {
        return view('welcome');
    });
});
```

Next create a `Dockerfile` like this:

```Dockerfile
FROM mumsnet/php-fpm-laravel:7.3

ARG COMPOSER_PARAMS_ARG=''
ENV COMPOSER_PARAMS=$COMPOSER_PARAMS_ARG

COPY --chown=www-data:www-data testlaravel /var/www/html
RUN composer install $COMPOSER_PARAMS && chown -R www-data:www-data /var/www/html

# No need for a final CMD.  It is already part of the base image.
# If you need to run db migrations etc before starting php/nginx, you can uncomment this
# CMD php artisan migrate; /usr/bin/supervisord -c /etc/supervisor/conf.d/supervisord.conf
```

Then create your `docker-compose.yml` like this:

```yaml
version: '3'

services:
  web:
    container_name: testlaravel
    build: .
    volumes:
      - ./testlumen1:/var/www/html
networks:
  default:
    external:
      name: mn_network
```

Your project folder structure should end up looking something like this, with your entire 
Lumen app self-contained in a subfolder:

![Folder structure](folders.png)

Finally add the relevant nginx entry to your router service `dev_server.conf` file:

```nginx
location ~ ^/(service/testlaravel) {
    proxy_set_header Host ${MY_HOSTNAME};
    proxy_pass http://testlaravel:8080;
    include proxy_directives.conf;
}
```

Start your new testlaravel service and restart the router service and you should be able to access it:

https://lhYOURNAME.devmn.net/service/testlaravel
