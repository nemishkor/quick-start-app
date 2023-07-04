FROM php:8.2-fpm-alpine3.17

RUN apk add --update --no-cache \
    curl \
    nano \
    git \
    # for usermod and groupmod
    shadow \
    # for envsubst command
    gettext \
    # to run build as www-data user
    sudo \
    supervisor \
    sqlite; \
    rm -rf /var/cache/apk/*

COPY conf.d/php.ini /usr/local/etc/php/php.ini

RUN docker-php-ext-install -j$(nproc) opcache
ARG PHP_OPCACHE_ENABLE="1"
COPY conf.d/opcache.ini /usr/local/etc/php/conf.d/opcache.ini.template
RUN export PHP_OPCACHE_ENABLE=$PHP_OPCACHE_ENABLE; \
    envsubst < /usr/local/etc/php/conf.d/opcache.ini.template > /usr/local/etc/php/conf.d/40-opcache.ini
RUN rm -f /usr/local/etc/php/conf.d/opcache.ini.template

# install gnu-libiconv and set LD_PRELOAD env to make iconv work fully on Alpine image.
# see https://github.com/docker-library/php/issues/240#issuecomment-763112749
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so

# Enable php8-xdebug if $PHP_XDEBUG_MODE is not empty
ARG PHP_XDEBUG_MODE=off
ARG PHP_XDEBUG_CLIENT_PORT=5902
ARG PHP_XDEBUG_CLIENT_HOST=host.docker.internal
ARG PHP_IDE_CONFIG="serverName=SomeName"
COPY conf.d/xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini.template
RUN if [[ "$PHP_XDEBUG_MODE" != "" ]]; then \
        apk add --no-cache php82-pecl-xdebug; \
        export PHP_XDEBUG_MODE=$PHP_XDEBUG_MODE; \
        export PHP_XDEBUG_CLIENT_PORT=$PHP_XDEBUG_CLIENT_PORT; \
        export PHP_XDEBUG_CLIENT_HOST=$PHP_XDEBUG_CLIENT_HOST; \
        envsubst < /usr/local/etc/php/conf.d/xdebug.ini.template > /usr/local/etc/php/conf.d/50-xdebug.ini; \
    fi; \
    rm -f /usr/local/etc/php/conf.d/xdebug.ini.template

COPY --from=composer /usr/bin/composer /usr/bin/composer

RUN apk add --no-cache bash; \
    curl -1sLf 'https://dl.cloudsmith.io/public/symfony/stable/setup.alpine.sh' | sudo -E bash; \
    apk add symfony-cli; \
    rm -rf /var/cache/apk/*

ARG UID=82
ARG GID=82
RUN set -x; \
    usermod -u $UID www-data; \
    groupmod -g $GID www-data

ARG AUTOSTART_APP_DAEMONS=true
COPY supervisor.conf /etc/supervisor/supervisor.conf.template
RUN export AUTOSTART_APP_DAEMONS=$AUTOSTART_APP_DAEMONS; \
    envsubst < /etc/supervisor/supervisor.conf.template > /etc/supervisor/supervisor.conf; \
    rm /etc/supervisor/supervisor.conf.template

RUN apk add --no-cache python3;  \
    rm -rf /var/cache/apk/*; \
    curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl; \
    chmod a+rx /usr/local/bin/youtube-dl; \
    youtube-dl -U; \
    curl -L https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp -o /usr/local/bin/yt-dlp; \
    chmod a+rx /usr/local/bin/yt-dlp; \
    yt-dlp -U

ARG APP_ENV=prod
ENV APP_ENV $APP_ENV
ARG COMPOSER_NO_DEV
ENV COMPOSER_NO_DEV $COMPOSER_NO_DEV

WORKDIR /var/www/app

COPY docker-entrypoint.sh /docker-entrypoint.sh
RUN chmod u+x /docker-entrypoint.sh

ENTRYPOINT ["/docker-entrypoint.sh"]
