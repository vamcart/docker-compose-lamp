FROM php:7.3-apache-stretch

# Surpresses debconf complaints of trying to install apt packages interactively
# https://github.com/moby/moby/issues/4032#issuecomment-192327844
 
ARG DEBIAN_FRONTEND=noninteractive

# Remove the old sources.list file.
RUN rm /etc/apt/sources.list

# Fix the source.list for stretch
RUN printf "deb http://archive.debian.org/debian/ stretch main\n" > /etc/apt/sources.list && \
    printf "deb-src http://archive.debian.org/debian/ stretch main\n" >>  /etc/apt/sources.list && \
    printf "deb http://archive.debian.org/debian-security stretch/updates main\n" >>  /etc/apt/sources.list && \
    printf "deb-src http://archive.debian.org/debian-security stretch/updates main" >>  /etc/apt/sources.list

# Update
RUN apt-get -y update --fix-missing && \
    apt-get upgrade -y && \
    apt-get --no-install-recommends install -y apt-utils && \
    rm -rf /var/lib/apt/lists/*


# Install useful tools and install important libaries
RUN apt-get -y update && \
    apt-get -y --no-install-recommends install nano wget dialog libsqlite3-dev libsqlite3-0 && \
    apt-get -y --no-install-recommends install mysql-client zlib1g-dev libzip-dev libicu-dev && \
    apt-get -y --no-install-recommends install --fix-missing apt-utils build-essential git curl && \ 
    apt-get -y --no-install-recommends install --fix-missing libcurl3 libcurl3-dev zip openssl && \
    rm -rf /var/lib/apt/lists/* && \
    curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Install xdebug
RUN pecl install xdebug-2.7.2 && \
    docker-php-ext-enable xdebug

# Install redis
RUN pecl install redis-5.0.2 && \
    docker-php-ext-enable redis

# Other PHP7 Extensions

RUN docker-php-ext-install pdo_mysql && \
    docker-php-ext-install pdo_sqlite && \
    docker-php-ext-install bcmath && \
    docker-php-ext-install mysqli && \
    docker-php-ext-install curl && \
    docker-php-ext-install tokenizer && \
    docker-php-ext-install json && \
    docker-php-ext-install zip && \
    docker-php-ext-install -j$(nproc) intl && \
    docker-php-ext-install mbstring && \
    docker-php-ext-install gettext

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
    
# Install Freetype 
RUN apt-get -y update && \
    apt-get --no-install-recommends install -y libfreetype6-dev libjpeg62-turbo-dev libpng-dev && \
    rm -rf /var/lib/apt/lists/* && \
    docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ && \
    docker-php-ext-install -j$(nproc) gd

# Insure an SSL directory exists
RUN mkdir -p /etc/apache2/ssl

# Enable SSL support
RUN a2enmod ssl && a2enmod rewrite

# Enable apache modules
RUN a2enmod rewrite headers

# Cleanup
RUN rm -rf /usr/src/*
