FROM php:8.1-fpm

ARG APP_ENV

ARG DOCKER_CONTAINER_UID
ARG DOCKER_CONTAINER_GID

RUN usermod  --uid $DOCKER_CONTAINER_UID www-data
RUN groupmod --gid $DOCKER_CONTAINER_GID www-data

RUN apt-get update && apt-get install -y --no-install-recommends \
    libzip-dev libpng-dev libfreetype6-dev libjpeg62-turbo-dev libmemcached-dev nano build-essential cron mcrypt

RUN docker-php-ext-install -j$(nproc) pdo_mysql zip exif pcntl gd # mbstring curl intl
#bz2 gettext iconv mysqli
RUN apt-get install -y libpq-dev \
    && docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql \
    && docker-php-ext-install pdo pdo_pgsql pgsql

RUN pecl install memcached && docker-php-ext-enable memcached

RUN apt-get install -y --no-install-recommends libssl-dev npm
RUN pecl install mongodb && docker-php-ext-enable mongodb

RUN echo 'max_execution_time = 300' >> "$PHP_INI_DIR/php.ini"
RUN echo 'max_execution_time = 300' >> /usr/local/etc/php/conf.d/docker-php-maxexectime.ini

RUN pecl install mailparse
RUN pecl install xdebug && docker-php-ext-enable xdebug
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

COPY cron/laravel-cron /etc/cron.d/laravel_cron
RUN chmod 0644 /etc/cron.d/laravel_cron \
    && crontab /etc/cron.d/laravel_cron \
    && touch /var/log/cron.log

RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"
RUN if [ "$APP_ENV" = local ]; then mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"; fi
RUN echo "memory_limit = 256M" >> "$PHP_INI_DIR/php.ini"
RUN echo "upload_max_filesize = 128M" >> "$PHP_INI_DIR/php.ini"
RUN echo "extension=mailparse.so" >> "$PHP_INI_DIR/php.ini"

CMD ["php-fpm"]
