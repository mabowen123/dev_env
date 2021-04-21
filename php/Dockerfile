FROM php:7.0-fpm

ARG java
ARG xdebug
ARG hprose

# ENV http_proxy http://host.docker.internal:1080
# ENV https_proxy http://host.docker.internal:1080

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN sed -i "s@http://deb.debian.org@http://mirrors.aliyun.com@g" /etc/apt/sources.list && rm -Rf /var/lib/apt/lists/*

# 安装 java（后台打包 apk）
RUN if [ "$java" = "True" ] ; then mkdir -p /usr/share/man/man1 \
  && apt-get update \
  && apt-get install -y openjdk-11-jdk; \
  fi

# PHP用户
RUN usermod -u 1000 www-data

RUN apt-get update && apt-get install -y \
  libfreetype6-dev \
  libjpeg62-turbo-dev \
  libpng-dev \
  libmemcached-dev \
  libsasl2-dev \
  wget \
  zip \
  unzip \
  libmcrypt-dev \
  libgmp-dev \
  libssl-dev \
  openssl \
  vim \
  cron \
  supervisor \
  && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
  && docker-php-ext-install -j$(nproc) gd \
  && docker-php-ext-install bcmath \
  && docker-php-ext-install gmp \
  && docker-php-ext-install mcrypt \
  && docker-php-ext-install sockets\
  && docker-php-ext-install zip \
  && docker-php-ext-install pdo_mysql \
  && docker-php-ext-install opcache \
  && docker-php-ext-install mysqli \
  && pecl channel-update pecl.php.net \
  && pecl install mongodb \
  && pecl install redis-3.1.6 \
  && docker-php-ext-enable mongodb redis \
  && export EDITOR=vim

# xdebug
RUN if [ "$xdebug" = "True" ] ; then pecl install xdebug \
  && docker-php-ext-enable xdebug; \
  fi

# hprose
RUN if [ "$hprose" = "True" ] ; then pecl install swoole hprose \
  && docker-php-ext-enable swoole hprose; \
  fi

# 安装 composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
  && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
  && php -r "unlink('composer-setup.php');" \
  && chown -R www-data:www-data /var/www/

USER www-data
RUN composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/ \
  && composer self-update

WORKDIR /workspace/html
