FROM php:7.3-fpm-alpine

ARG PHP_TIMEZONE
ARG PHP_MEMORY_LIMIT
ARG USER_ID

RUN apk add --update --no-cache openssl unzip libzip-dev autoconf inkscape

# Install PHP extensions
RUN docker-php-ext-install -j$(nproc) pdo pdo_mysql opcache zip

RUN apk add --no-cache icu-dev && docker-php-ext-configure intl && docker-php-ext-install intl

RUN pecl install xdebug imagick redis mongodb

RUN docker-php-ext-enable xdebug imagick redis mongodb

RUN apk add --no-cache librdkafka-dev && pecl install rdkafka && docker-php-ext-enable rdkafka

# Install Blackfire
# RUN version=$(php -r "echo PHP_MAJOR_VERSION.PHP_MINOR_VERSION;") \
#     && curl -A "Docker" -o /tmp/blackfire-probe.tar.gz -D - -L -s https://blackfire.io/api/v1/releases/probe/php/linux/amd64/$version \
#     && tar zxpf /tmp/blackfire-probe.tar.gz -C /tmp \
#     && mv /tmp/blackfire-*.so $(php -r "echo ini_get('extension_dir');")/blackfire.so \
#     && printf "extension=blackfire.so\nblackfire.agent_socket=tcp://blackfire:8707\n" > $PHP_INI_DIR/conf.d/blackfire.ini    

# Set timezone
RUN ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo Asia/Shanghai > /etc/timezone

# Install composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
RUN php composer-setup.php
RUN php -r "unlink('composer-setup.php');"
RUN mv composer.phar /usr/local/bin/composer
COPY composer/config.json /var/www/.composer/config.json

# Copy xdebug and php config.
COPY conf.d/* /usr/local/etc/php/conf.d/

# Map user id from host user when it's provided
RUN if [ ! -z ${USER_ID} ]; then usermod -u ${USER_ID} www-data; fi
RUN if [ ! -z ${USER_ID} ]; then groupmod -g ${USER_ID} www-data; fi

# set default user and working directory
USER www-data
WORKDIR /var/www/project
