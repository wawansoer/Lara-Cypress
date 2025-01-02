FROM php:8.3.6-cli-alpine3.18

COPY --from=composer:2.7.4 /usr/bin/composer /usr/bin/

RUN \
    set -ex && \
    apk update && \
    apk add --no-cache libstdc++ libpq && \
    apk add --no-cache --virtual .build-deps $PHPIZE_DEPS curl-dev linux-headers postgresql-dev openssl-dev pcre-dev pcre2-dev zlib-dev && \
    apk add --no-cache supervisor && \
    apk add --no-cache supercronic && \
    pecl channel-update pecl.php.net && \
    docker-php-ext-install sockets && \
    pecl install redis &&  \
    docker-php-ext-enable redis && \
    docker-php-source extract && \
    mkdir /usr/src/php/ext/swoole && \
    curl -sfL https://github.com/swoole/swoole-src/archive/v5.1.2.tar.gz -o swoole.tar.gz && \
    tar xfz swoole.tar.gz --strip-components=1 -C /usr/src/php/ext/swoole && \
    docker-php-ext-configure swoole \
    --enable-swoole-pgsql \
    --enable-openssl      \
    --enable-sockets --enable-swoole-curl && \
    docker-php-ext-install -j$(nproc) swoole && \
    docker-php-ext-configure pcntl --enable-pcntl && \
    docker-php-ext-install pcntl && \
    rm -f swoole.tar.gz && \
    docker-php-source delete && \
    apk del .build-deps

# TODO not use node in producation
RUN apk add --no-cache nodejs npm git

COPY docker/php/supervisord/supervisord.conf /etc/

RUN mkdir -m 446 /.composer /.npm

EXPOSE 8000

WORKDIR "/var/www/"

ENTRYPOINT ["supervisord", "--nodaemon", "--configuration", "/etc/supervisord.conf"]
