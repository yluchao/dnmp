FROM alpine:3.9

ENV PHPIZE_DEPS \
		autoconf \
		dpkg-dev dpkg \
		file \
		g++ \
		gcc \
		libc-dev \
		make \
		pkgconf \
		re2c

# persistent / runtime deps
RUN apk add --no-cache \
		ca-certificates \
		curl \
		tar \
		xz \
		openssl \
		unzip

# 设置用户
RUN set -x \
	&& addgroup -g 82 -S www \
	&& adduser -u 82 -D -S -G www www

ENV PHP_INI_DIR /usr/local/etc/php
RUN set -eux; \
	mkdir -p "$PHP_INI_DIR/conf.d"; \
	[ ! -d /var/www/html ]; \
	mkdir -p /var/www/html; \
	chown www:www /var/www/html; \
	chmod 777 /var/www/html; \
	mkdir -p /usr/src

ENV PHP_EXTRA_CONFIGURE_ARGS --enable-fpm --with-fpm-user=www --with-fpm-group=www --disable-cgi

ENV PHP_CFLAGS="-fstack-protector-strong -fpic -fpie -O2"
ENV PHP_CPPFLAGS="$PHP_CFLAGS"
ENV PHP_LDFLAGS="-Wl,-O1 -Wl,--hash-style=both -pie"

COPY docker-php-source /usr/local/bin/
COPY php.tar.xz /usr/src/
COPY ext/* /usr/src/
COPY docker-php-ext-* docker-php-entrypoint /usr/local/bin/

RUN set -xe \
	&& apk add --no-cache --virtual .build-deps \
		$PHPIZE_DEPS \
		coreutils \
		curl-dev \
		libedit-dev \
		libxml2-dev \
		libjpeg-turbo-dev \
		freetype-dev \
		gd-dev gettext-dev \
		libxpm-dev \
		libpng-dev \
		libmcrypt-dev \
		openssl-dev \
		sqlite-dev \
		libsodium-dev \
		sqlite-dev \
		yaml-dev \
		rabbitmq-c-dev \
	\
	&& apk add tzdata \
	&& ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
	&& echo "Asia/Shanghai" > /etc/timezone \
	&& export CFLAGS="$PHP_CFLAGS" \
		CPPFLAGS="$PHP_CPPFLAGS" \
		LDFLAGS="$PHP_LDFLAGS" \
	&& docker-php-source extract \
	&& cd /usr/src/php \
	&& gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
	&& ./configure \
		--build="$gnuArch" \
		--with-config-file-path="$PHP_INI_DIR" \
		--with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
		--enable-option-checking=fatal \
		--with-mhash \
		--enable-ftp \
		--enable-mbstring \
		--enable-mysqlnd \
		--enable-opcache \
    	--enable-pcntl \
		--enable-xml \
		--enable-sockets \
		--with-curl \
		--with-gd=/usr \
	    --with-jpeg-dir \
	    --with-freetype-dir \
	    --with-xpm-dir \
	    --with-png-dir \
	    --with-gettext \
	    --with-iconv \
	    --with-libxml-dir=/usr \
    	--with-mcrypt \
	    --with-png-dir \
    	--with-gettext \
		--with-libedit \
		--with-mysqli=mysqlnd \
    	--with-pdo-mysql=mysqlnd \
		--with-openssl \
		--with-zlib \
		$(test "$gnuArch" = 's390x-linux-gnu' && echo '--without-pcre-jit') \
		$PHP_EXTRA_CONFIGURE_ARGS \
	&& make -j "$(nproc)" \
	&& find -type f -name '*.a' -delete \
	&& make install \
	&& { find /usr/local/bin /usr/local/sbin -type f -perm +0111 -exec strip --strip-all '{}' + || true; } \
	&& make clean \
	&& cp -v php.ini-* "$PHP_INI_DIR/" \
	&& cp -v php.ini-production "$PHP_INI_DIR/php.ini" \
	&& sed -i "s@^memory_limit.*@memory_limit = 256M@" $PHP_INI_DIR/php.ini \
	&& sed -i 's@^output_buffering =@output_buffering = On\noutput_buffering =@' $PHP_INI_DIR/php.ini \
	&& sed -i 's@^;cgi.fix_pathinfo.*@cgi.fix_pathinfo=0@' $PHP_INI_DIR/php.ini \
	&& sed -i 's@^short_open_tag = Off@short_open_tag = On@' $PHP_INI_DIR/php.ini \
	&& sed -i 's@^expose_php = On@expose_php = Off@' $PHP_INI_DIR/php.ini \
	&& sed -i 's@^request_order.*@request_order = "CGP"@' $PHP_INI_DIR/php.ini \
	&& sed -i 's@^;date.timezone.*@date.timezone = Asia/Shanghai@' $PHP_INI_DIR/php.ini \
	&& sed -i 's@^post_max_size.*@post_max_size = 1024M@' $PHP_INI_DIR/php.ini \
	&& sed -i 's@^upload_max_filesize.*@upload_max_filesize = 1024M@' $PHP_INI_DIR/php.ini \
	&& sed -i 's@^max_execution_time.*@max_execution_time = 600@' $PHP_INI_DIR/php.ini \
	&& sed -i 's@^;realpath_cache_size.*@realpath_cache_size = 2M@' $PHP_INI_DIR/php.ini \
	&& [ -e /usr/sbin/sendmail ] && sed -i 's@^;sendmail_path.*@sendmail_path = /usr/sbin/sendmail -t -i@' $PHP_INI_DIR/php.ini \
	&& sed -i "s@^;curl.cainfo.*@curl.cainfo = ${openssl_install_dir}/cert.pem@" $PHP_INI_DIR/php.ini \
	&& sed -i "s@^;openssl.cafile.*@openssl.cafile = ${openssl_install_dir}/cert.pem@" $PHP_INI_DIR/php.ini \
	# php 7.2以下需要手动安装
	&& pecl install /usr/src/libsodium-2.0.21.tgz \
	&& pecl install /usr/src/yaml-2.0.4.tgz \
	&& pecl install /usr/src/redis-4.3.0.tgz \
	&& pecl install /usr/src/mongodb-1.5.3.tgz \
	&& pecl install /usr/src/amqp-1.9.4.tgz \
	&& docker-php-ext-enable sodium yaml redis mongodb amqp \
    && docker-php-ext-install zip \
    && tar -zxvf /usr/src/memcache.tgz \
    && cd /usr/src/memcache \
    && phpize \
    && ./configure \
    && make \
    && make install \
    && echo "extension=memcache.so" > /usr/local/etc/php/conf.d/memcache.ini \
	&& cd / \
	&& docker-php-source delete \
	\
	&& runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)" \
	&& apk add --no-cache $runDeps \
	\
	&& apk del --no-network .build-deps \
	&& pecl update-channels \
	&& rm -rf /tmp/pear ~/.pearrc /usr/src/*

# 安装扩展
# Run set -ex \
#	&& apk add --no-cache postgresql-dev && docker-php-ext-install pgsql pdo_pgsql


ENTRYPOINT ["docker-php-entrypoint"]
##<autogenerated>##
WORKDIR /var/www/html

# 配置php-fpm
RUN set -ex \
	&& cd /usr/local/etc \
	&& if [ -d php-fpm.d ]; then \
		# for some reason, upstream's php-fpm.conf.default has "include=NONE/etc/php-fpm.d/*.conf"
		sed 's!=NONE/!=!g' php-fpm.conf.default | tee php-fpm.conf > /dev/null; \
		cp php-fpm.d/www.conf.default php-fpm.d/www.conf; \
	else \
		# PHP 5.x doesn't use "include=" by default, so we'll create our own simple config that mimics PHP 7+ for consistency
		mkdir php-fpm.d; \
		cp php-fpm.conf.default php-fpm.d/www.conf; \
		{ \
			echo '[global]'; \
			echo 'include=etc/php-fpm.d/*.conf'; \
		} | tee php-fpm.conf; \
	fi \
	&& { \
		echo '[global]'; \
		echo 'error_log = /proc/self/fd/2'; \
		echo; \
		echo '[www]'; \
		echo '; if we send this to /proc/self/fd/1, it never appears'; \
		echo 'access.log = /proc/self/fd/2'; \
		echo; \
		echo 'clear_env = no'; \
		echo; \
		echo '; Ensure worker stdout and stderr are sent to the main error log.'; \
		echo 'catch_workers_output = yes'; \
	} | tee php-fpm.d/docker.conf \
	&& { \
		echo '[global]'; \
		echo 'daemonize = no'; \
		echo; \
		echo '[www]'; \
		echo 'listen = 9000'; \
	} | tee php-fpm.d/zz-docker.conf

EXPOSE 9000
CMD ["php-fpm"]
