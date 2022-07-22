FROM php:8.0-apache

LABEL vendor="Mautic"

LABEL maintainer="Luiz Eduardo Oliveira Fonseca <luiz@powertic.com>"

# Install PHP extensions
RUN apt-get update && apt-get install --no-install-recommends -y \
    cron \
    git \
    wget \
    sudo \
    libc-client-dev \
    libicu-dev \
    libkrb5-dev \
    libmcrypt-dev \
    libssl-dev \
    libzip-dev \
    libz-dev \
    unzip \
    zip \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    && rm -rf /var/lib/apt/lists/* \
    && rm /etc/cron.daily/*

RUN pecl install mcrypt-1.0.4 \
    && docker-php-ext-configure imap --with-imap --with-imap-ssl --with-kerberos \
    && docker-php-ext-configure opcache --enable-opcache \
    && docker-php-ext-install imap intl mysqli pdo_mysql zip opcache bcmath\
    && docker-php-ext-enable imap intl mcrypt mysqli pdo_mysql zip opcache bcmath

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer

# Define Mautic volume to persist data
VOLUME /data

# Define Mautic version and expected SHA1 signature
ARG MAUTIC_VERSION=4.4.0
ENV MAUTIC_VERSION $MAUTIC_VERSION
ARG MAUTIC_SHA1=5a19eb5186ae80a4bcff892607581e5569686fbf
ENV MAUTIC_SHA1 $MAUTIC_SHA1

# By default enable cron jobs
ENV MAUTIC_RUN_CRON_JOBS true

# By default disable automatic migrations
ENV MAUTIC_RUN_MIGRATIONS false

# Setting an Default database user for Mysql
ENV MAUTIC_DB_USER root

# Setting an Default database name for Mysql
ENV MAUTIC_DB_NAME mautic

# Set up mysql port
ENV MAUTIC_DB_PORT 3306

# Setting PHP properties
ENV PHP_INI_DATE_TIMEZONE='UTC' \
    PHP_MEMORY_LIMIT=512M \
    PHP_MAX_UPLOAD=512M \
    PHP_MAX_EXECUTION_TIME=300

# Download package and extract to web volume
RUN curl -o /tmp/mautic.zip -SL https://github.com/mautic/mautic/releases/download/${MAUTIC_VERSION}/${MAUTIC_VERSION}.zip \
    && echo "$MAUTIC_SHA1 */tmp/mautic.zip" | sha1sum -c - \
    && rm -rf /var/www/html \
    && mkdir /var/www/html \
    && unzip /tmp/mautic.zip -d /var/www/html \
    && rm /tmp/mautic.zip \
    && chown -R www-data:www-data /var/www/html

# Drop in redirectors to volume
COPY common/paths_local.php /var/www/html/app/config/paths_local.php
COPY common/parameters_local.php /var/www/html/app/config/parameters_local.php

# Copy init scripts and custom .htaccess
COPY common/docker-entrypoint.sh /entrypoint.sh
COPY common/makeconfig.php /makeconfig.php
COPY common/makedb.php /makedb.php
COPY common/mautic.crontab /etc/cron.d/mautic
RUN chmod 644 /etc/cron.d/mautic

# Enable Apache Rewrite Module
RUN a2enmod rewrite

# Apply necessary permissions
RUN ["chmod", "+x", "/entrypoint.sh"]
ENTRYPOINT ["/entrypoint.sh"]

CMD ["apache2-foreground"]
