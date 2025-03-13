ARG ALPINE_VERSION=3.21
FROM alpine:${ALPINE_VERSION}
LABEL Maintainer="github.com/EdyTheCow"
LABEL Description="Alpine Linux based container running Paymenter"

WORKDIR /var/www/html

# Install packages
RUN apk add --no-cache \
  curl \
  nginx \
  tar \
  unzip \
  git \
  ca-certificates \
  supercronic \
  gnupg \
  php84 \
  php84-common \
  php84-curl \
  php84-fpm \
  php84-gd \
  php84-intl \
  php84-mbstring \
  php84-xml \
  php84-pdo_mysql \
  php84-bcmath \
  php84-pecl-redis \
  php84-zip \
  php84-phar \
  php84-iconv \
  php84-dom \
  php84-fileinfo \
  php84-sodium \
  php84-tokenizer \
  php84-xmlreader \
  supervisor

# Create alias for php
RUN ln -s /usr/bin/php84 /usr/bin/php

# Configure nginx - http
COPY config/nginx.conf /etc/nginx/nginx.conf
# Configure nginx - default server
COPY config/conf.d /etc/nginx/conf.d/

# Configure PHP-FPM
ENV PHP_INI_DIR /etc/php84
COPY config/fpm-pool.conf ${PHP_INI_DIR}/php-fpm.d/www.conf
COPY config/php.ini ${PHP_INI_DIR}/conf.d/custom.ini

# Configure supervisord
COPY config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Add crontab for supercronic to run
COPY config/crontab /etc/crontab

# Install composer from the official image
COPY --from=composer /usr/bin/composer /usr/bin/composer

# Download paymenter
RUN curl -Lo paymenter.tar.gz https://api.paymenter.org/beta \
    && tar -xzvf paymenter.tar.gz \
    && rm paymenter.tar.gz \
    && chmod -R 755 storage/* bootstrap/cache/

# Install packages
RUN composer install --optimize-autoloader --no-interaction --no-progress --no-dev

# Make sure files/folders needed by the processes are accessable when they run under the nobody user
RUN chown -R nobody:nobody /var/www/html /run /var/lib/nginx /var/log/nginx

# Switch to use a non-root user from here on
USER nobody

# Expose the port nginx is reachable on
EXPOSE 8080

# Let supervisord start services
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

# Configure a healthcheck to validate that everything is up & running
HEALTHCHECK --timeout=10s CMD curl --silent --fail http://127.0.0.1:8080/fpm-ping || exit 1
