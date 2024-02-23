FROM php:8.3-fpm-bullseye AS base

WORKDIR /workspace

# timezone environment
ENV TZ=UTC \
  # locale
  LANG=en_US.UTF-8 \
  LANGUAGE=en_US:en \
  LC_ALL=en_US.UTF-8 \
  # composer environment
  COMPOSER_ALLOW_SUPERUSER=1 \
  COMPOSER_HOME=/composer

COPY --from=composer:2.6 /usr/bin/composer /usr/bin/composer

RUN  apt-get update
RUN  apt-get -y install --no-install-recommends \
    locales \
    git \
    unzip \
    libzip-dev \
    libicu-dev \
    libonig-dev
RUN  locale-gen en_US.UTF-8
RUN  localedef -f UTF-8 -i en_US en_US.UTF-8
RUN  docker-php-ext-install \
    intl \
    pdo_mysql \
    zip \
    bcmath
RUN  composer config -g process-timeout 3600
RUN  composer config -g repos.packagist composer https://packagist.org


FROM base AS development

RUN  apt-get -y install --no-install-recommends \
    default-mysql-client
RUN  apt-get clean
RUN  rm -rf /var/lib/apt/lists/*

COPY ./infra/docker/php/php.development.ini /usr/local/etc/php/php.ini

FROM development AS development-xdebug

RUN  pecl install xdebug
RUN  docker-php-ext-enable xdebug

COPY ./infra/docker/php/xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

FROM base AS deploy

COPY ./infra/docker/php/php.deploy.ini /usr/local/etc/php/php.ini
COPY ./src /workspace

RUN  composer install -q -n --no-ansi --no-dev --no-scripts --no-progress --prefer-dist
RUN  chmod -R 777 storage bootstrap/cache
RUN  php artisan optimize:clear
RUN  php artisan optimize
RUN  apt-get clean
RUN  rm -rf /var/lib/apt/lists/*
