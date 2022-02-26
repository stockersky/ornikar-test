FROM php:7-fpm

WORKDIR /composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
RUN php composer-setup.php
RUN php -r "unlink('composer-setup.php');"
RUN mv composer.phar /usr/local/bin/composer

WORKDIR /app
COPY composer.json composer.lock ./
RUN apt-get update && apt-get install -y git
RUN composer install

ADD public ./public
ENTRYPOINT ["composer", "run", "dev:start"]