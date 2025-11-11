FROM php:8.3-fpm

RUN apt update && apt install -y \
    vim \
    libicu-dev \
    zlib1g-dev \
    libzip-dev \
    libpng-dev \
    libxml2-dev \
    libxslt1-dev \
    libjpeg-dev \
    libfreetype6-dev

RUN docker-php-ext-install pdo_mysql \
    bcmath \
    intl \
    zip \
    soap \
    xsl \
    sockets \
    ftp

RUN docker-php-ext-configure gd --with-jpeg --with-freetype\
    && docker-php-ext-install -j$(nproc) gd

#increase memory limit
RUN cd /usr/local/etc/php/conf.d/ && \
  echo 'memory_limit = -1' >> /usr/local/etc/php/conf.d/docker-php-memlimit.ini

COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer
