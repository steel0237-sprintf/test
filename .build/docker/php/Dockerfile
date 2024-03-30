#syntax=docker/dockerfile:1.7-labs
FROM php:7.4-apache

RUN apt-get update && apt-get install -y tzdata && \
    rm /etc/localtime && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    dpkg-reconfigure -f noninteractive tzdata

COPY --exclude=.build --exclude=.git --chown=www-data:www-data ./ /var/www/sanatoriy/data/www/putevka.com

USER www-data
WORKDIR /var/www/sanatoriy/data/www/putevka.com

