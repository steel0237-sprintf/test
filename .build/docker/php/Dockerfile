#syntax=docker/dockerfile:1.7-labs
FROM php:7.4-apache

RUN apt-get update && apt-get install -y tzdata && \
    rm /etc/localtime && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    dpkg-reconfigure -f noninteractive tzdata

RUN a2enmod rewrite
RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"

RUN apt-get install -y locales wget curl libpq-dev libzip-dev zip unzip git libonig-dev libxml2-dev \
    libc-client-dev libkrb5-dev libpng-dev && \
    docker-php-ext-configure imap --with-kerberos --with-imap-ssl && \
    docker-php-ext-install -j$(nproc) iconv mbstring pdo_pgsql zip gettext intl sysvsem \
    imap gd sockets soap bcmath

RUN apt-get install -y libpng-dev libjpeg-dev libfreetype6-dev \
  && docker-php-ext-configure gd \
     --with-freetype=/usr/include/freetype2 \
     --with-jpeg=/usr/include \
  && docker-php-ext-install gd

RUN docker-php-ext-install pgsql

RUN echo 'ru_RU.UTF-8 UTF-8' >> /etc/locale.gen && \
    locale-gen
ENV LANG ru_RU.UTF-8
ENV LANGUAGE ru_RU:en
ENV LC_ALL ru_RU.UTF-8

RUN ln -s /usr/local/bin/php /bin/php

RUN wget https://getcomposer.org/installer && php installer && mv composer.phar /usr/local/bin/composer

COPY .build/docker/apache2/000-default.conf /etc/apache2/sites-available/000-default.conf

ENV TIMEZONE            Europe/Moscow
ENV TZ                  $TIMEZONE
ENV PHP_MEMORY_LIMIT    -1
ENV UPLOAD_MAX_FILESIZE 350M
ENV PHP_MAX_POST        350M
ENV ERROR_REPORTING     E_ALL & ~E_DEPRECATED & ~E_STRICT & ~E_NOTICE & ~E_WARNING
ENV PHP_MAX_INPUT_TIME        60
ENV PHP_MAX_EXECUTION_TIME   600

RUN echo "date.timezone = ${TIMEZONE}" >> $PHP_INI_DIR/php.ini && \
    echo "memory_limit = ${PHP_MEMORY_LIMIT}" >> $PHP_INI_DIR/php.ini && \
    echo "upload_max_filesize = ${UPLOAD_MAX_FILESIZE}" >> $PHP_INI_DIR/php.ini && \
    echo "post_max_size = ${PHP_MAX_POST}" >> $PHP_INI_DIR/php.ini && \
    echo "max_input_time = ${PHP_MAX_INPUT_TIME}" >> $PHP_INI_DIR/php.ini && \
    echo "max_execution_time = ${PHP_MAX_EXECUTION_TIME}" >> $PHP_INI_DIR/php.ini && \
    echo "error_reporting = ${ERROR_REPORTING}" >> $PHP_INI_DIR/php.ini

ARG BUILDUID
ARG BUILDGID
ARG PRIVATE=""

RUN usermod -u ${BUILDUID} www-data && \
    bash -c 'if [ `uname` == Linux ]; then groupmod -g ${BUILDGID} www-data; fi' && \
    mkdir /home/www-data /home/www-data/.ssh /home/www-data/.composer && \
    touch /home/www-data/.gitconfig && \
    usermod --home /home/www-data www-data && \
    chown -R www-data.www-data /home/www-data

COPY --exclude=.build --exclude=.git --chown=www-data:www-data ./ /var/www/sanatoriy/data/www/putevka.com

USER www-data
WORKDIR /var/www/sanatoriy/data/www/putevka.com

RUN echo "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config && chmod 700 ~/.ssh
#RUN eval $(ssh-agent -s) && bash  -c 'ssh-add <(echo "$PRIVATE")' && composer install --no-scripts
