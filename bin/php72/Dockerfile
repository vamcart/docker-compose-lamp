#Quickfix - Basebox for PHP7.2 Library now uses Debian "10" Buster, superceeding #libcurl3, stretch is most compatible at this time whilst devs workout backport.
#https://github.com/docker-library/php/issues/865

FROM php:7.2-apache-stretch

#Surpresses debconf complaints of trying to install apt packages interactively
#https://github.com/moby/moby/issues/4032#issuecomment-192327844

ARG DEBIAN_FRONTEND=noninteractive

# Fix the source.list for stretch
RUN printf "deb http://archive.debian.org/debian/ stretch main\n" > /etc/apt/sources.list && \
    printf "deb-src http://archive.debian.org/debian/ stretch main\n" >>  /etc/apt/sources.list && \
    printf "deb http://archive.debian.org/debian-security stretch/updates main\n" >>  /etc/apt/sources.list && \
    printf "deb-src http://archive.debian.org/debian-security stretch/updates main" >>  /etc/apt/sources.list

RUN apt-get -y update --fix-missing --no-install-recommends
RUN apt-get -y upgrade

# Install useful tools
RUN apt-get -yq install apt-utils nano wget dialog

# Install important libraries
RUN apt-get -y install --fix-missing apt-utils build-essential git curl libcurl3 libcurl3-dev zip openssl

# Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Install xdebug
RUN pecl install xdebug-2.6.0
RUN docker-php-ext-enable xdebug

# Install redis
RUN pecl install redis-4.0.1
RUN docker-php-ext-enable redis

# Other PHP7 Extensions

RUN apt-get -y install libsqlite3-dev libsqlite3-0 mysql-client
RUN docker-php-ext-install pdo_mysql 
RUN docker-php-ext-install pdo_sqlite
RUN docker-php-ext-install mysqli

RUN docker-php-ext-install curl
RUN docker-php-ext-install tokenizer
RUN docker-php-ext-install json
RUN docker-php-ext-install bcmath

RUN apt-get -y install zlib1g-dev
RUN docker-php-ext-install zip

RUN apt-get -y install libicu-dev
RUN docker-php-ext-install -j$(nproc) intl

RUN docker-php-ext-install mbstring
RUN docker-php-ext-install gettext

RUN apt-get install -y libfreetype6-dev libjpeg62-turbo-dev libpng-dev
RUN docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ 
RUN docker-php-ext-install -j$(nproc) gd

# Install IonCube

RUN ARCH=$(uname -m) && \
    PHP_VERSION=$(php -v | head -n 1 | awk '{ print $2 }') && \
    PHP_MAJOR_MINOR=$(echo "${PHP_VERSION}" | cut -d. -f1,2) && \
    IONCUBE_LOADER="ioncube_loader_lin_${PHP_MAJOR_MINOR}.so" && \
    if [ "$ARCH" = "x86_64" ]; then \
        IONBUDE_SRC="https://downloads.ioncube.com/loader_downloads/ioncube_loaders_lin_x86-64.tar.gz"; \
    elif [ "$ARCH" = "x86" ]; then \
        IONBUDE_SRC="https://downloads.ioncube.com/loader_downloads/ioncube_loaders_lin_x86.tar.gz"; \
    elif [ "$ARCH" = "armv7l" ]; then \
        IONBUDE_SRC="https://downloads.ioncube.com/loader_downloads/ioncube_loaders_lin_armv7l.tar.gz"; \
    elif [ "$ARCH" = "arm64" ]; then \
        IONBUDE_SRC="https://downloads.ioncube.com/loader_downloads/ioncube_loaders_dar_arm64.tar.gz"; \
    elif [ "$ARCH" = "i686" ]; then \
        IONBUDE_SRC="https://downloads.ioncube.com/loader_downloads/ioncube_loaders_lin_x86.tar.gz"; \
    else \
        echo "Unsupported architecture"; exit 1; \
    fi && \
    cd /usr/src && \
    curl -o ioncube.tar.gz ${IONBUDE_SRC} && \
    tar -xvvzf ioncube.tar.gz && \
    cp ioncube/${IONCUBE_LOADER} /usr/local/lib/php/extensions/* && \
    rm -Rf ioncube.tar.gz ioncube && \
    echo "zend_extension=${IONCUBE_LOADER}" > /usr/local/etc/php/conf.d/docker-php-ext-ioncube.ini
    
# Insure an SSL directory exists
RUN mkdir -p /etc/apache2/ssl

# Enable SSL support
RUN a2enmod ssl && a2enmod rewrite

# Enable apache modules
RUN a2enmod rewrite headers