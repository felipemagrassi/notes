# Multistage Building - 2023-05-16 21:24:08.668563222 -0300 -03 m=+420.830264783

#docker

Multistage building is a feature that allows you to build
a docker image in multiple steps.

This is useful when you want to build a docker image that is based on another image,
but you don't want to include all the build dependencies in the final image.

For example a react app that uses webpack to build the final bundle.

php example

```dockerfile
FROM php:7.4-cli AS builder

WORKDIR /var/www

RUN apt-get update && \
    apt-get install libzip-dev -y && \
    docker-php-ext-install zip

RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
    php composer-setup.php && \
    php -r "unlink('composer-setup.php');"

RUN php composer.phar create-project --prefer-dist laravel/laravel laravel

ENTRYPOINT ["php", "laravel/artisan", "serve"]

CMD ["--host=0.0.0.0"]

FROM php:7.4-fpm-alpine
WORKDIR /var/www
RUN rm -rf /var/www/html

COPY --from=builder /var/www/laravel .

RUN chown -R www-data:www-data /var/www

EXPOSE 9000

CMD ["php-fpm"]
```




