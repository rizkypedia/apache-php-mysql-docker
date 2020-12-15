FROM php:7.3-apache-buster

ARG APACHE_DOCUMENT_ROOT=/var/www/project/public
ARG PHP_MEMORY_LIMIT=2G
ARG PROJECT_NAME=app

ENV COMPOSER_ALLOW_SUPERUSER=1
ENV EXT_APCU_VERSION=5.1.17


ENV NVM_DIR $HOME/.nvm
ENV NODE_VERSION 10

# Configure apache
RUN sed -ri -e 's!/var/www/html!"${APACHE_DOCUMENT_ROOT}"!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!\#ServerName example.com!'"${PROJECT_NAME}"'.${DOMAIN_SUFFIX}:80!g'  /etc/apache2/sites-*/*.conf

# Install development tools as well as additional packages and do a cleanup afterwards
RUN apt-get update && apt-get install -y --no-install-recommends \
        git \
        ssh \
        zip \
        wget \
        default-mysql-client \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        zlib1g-dev \
        libzip-dev \
        apt-utils \
        libxml2-dev \
    && rm -rf /var/lib/apt/lists/* \
    && docker-php-ext-install -j$(nproc) iconv \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd mysqli pdo pdo_mysql zip soap opcache\
    && docker-php-ext-enable gd mysqli pdo pdo_mysql zip soap opcache \
    && pecl install xdebug-2.9.2 \
    && docker-php-ext-enable xdebug \
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
    && wget https://get.symfony.com/cli/installer -O - | bash \
    && docker-php-source delete \
    && pecl clear-cache \
    && rm -rf \
        /usr/include/php \
        /usr/lib/php/build \
        /tmp/* \
        /root/.composer \
        /var/cache/apk/*;


RUN docker-php-source extract \
    # ext-apcu
    && mkdir -p /usr/src/php/ext/apcu \
    && curl -fsSL https://github.com/krakjoe/apcu/archive/v$EXT_APCU_VERSION.tar.gz | tar xvz -C /usr/src/php/ext/apcu --strip 1 \
    && docker-php-ext-install apcu \
    && docker-php-ext-enable apcu \
    # cleanup
    && docker-php-source delete

RUN mkdir -p $NVM_DIR \
    && curl https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash

# Configure environment
COPY build/app/php.prod.ini /usr/local/etc/php/conf.d/local.ini
RUN a2enmod rewrite; \
    echo "export PATH=\"/var/www/project/bin:${PATH}\"" > ~/.bashrc;


COPY ./project /var/www/project

RUN touch /var/www/project/.env; \
    mkdir -p /var/www/project/var; \
    chmod 0777 /var/www/project/var; \
    chown www-data:www-data /var/www/project/var

WORKDIR /var/www/project
RUN composer install --no-dev --no-scripts
