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

# Create entrypoint script to fix permissions
RUN echo '#!/bin/bash\n\
if [ -d /var/www/html ]; then\n\
  find /var/www/html -type f -exec chmod 644 {} +\n\
  find /var/www/html -type d -exec chmod 755 {} +\n\
  chmod 775 /var/www/html/var\n\
  chmod 775 /var/www/html/pub\n\
  chmod 775 /var/www/html/generated\n\
  chmod 775 /var/www/html/app/etc 2>/dev/null || true\n\
  find /var/www/html/bin -type f -exec chmod +x {} + 2>/dev/null || true\n\
  find /var/www/html/vendor/bin -type f -exec chmod +x {} + 2>/dev/null || true\n\
  sed -i "s/^root \$MAGE_ROOT\/pub;/#root \$MAGE_ROOT\/pub;/" /var/www/html/nginx.conf.sample 2>/dev/null || true\n\
fi\n\
exec docker-php-entrypoint "$@"' > /usr/local/bin/entrypoint.sh && \
chmod +x /usr/local/bin/entrypoint.sh

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
CMD ["php-fpm"]
