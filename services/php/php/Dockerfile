FROM php:7.3-fpm
ADD ./sources.list /etc/apt
RUN apt-get update \
    # 相关依赖必须手动安装
    && apt-get install -y --no-install-recommends \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpq-dev \
        g++ \
        libicu-dev \
        libxml2-dev \
        libpng-dev \
        git \
        vim \
        procps \
        unzip \
    # 安装扩展
    && docker-php-ext-install -j$(nproc) iconv mcrypt \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-install mbstring \
    && docker-php-ext-install zip \
    && docker-php-ext-install pdo pdo_mysql mysqli opcache \
    && docker-php-ext-install pcntl \
    && docker-php-ext-install sockets \
    #&& pecl install xdebug \
    #&& docker-php-ext-enable xdebug \ 使用swoole时需要关闭 xdebug或者修改端口号
    && pecl install redis \
    && docker-php-ext-enable redis \
    && pecl install swoole \
    && docker-php-ext-enable swoole \
    && git clone https://github.com/websupport-sk/pecl-memcache.git memcache \
    && cd memcache \
    && phpize \
    && ./configure \
    && make \
    && make install \
    && echo "extension=memcache.so" > /usr/local/etc/php/conf.d/memcache.ini \
#    && php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
#    && php composer-setup.php \
#    && php -r "unlink('composer-setup.php');" \
#    && mv composer.phar /usr/local/bin/composer \
#    && composer self-update \
    && rm -rf memcache \
    && rm -rf /var/lib/apt/lists/*

