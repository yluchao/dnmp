FROM leleos/php-fpm:5.6

RUN set -x \
    \
    && buildDeps=' \
        ca-certificates \
        dpkg-dev \
        gcc \
        libc6-dev \
        libevent-dev \
        libsasl2-dev \
        make \
        perl \
        wget \
    ' \
    && apt-get update && apt-get install -y $buildDeps --no-install-recommends \
    && wget -O memcached.tar.gz "https://memcached.org/files/memcached-1.5.14.tar.gz" \
    && mkdir -p /usr/src/memcached \
    && tar -xzf memcached.tar.gz -C /usr/src/memcached --strip-components=1 \
    && rm memcached.tar.gz \
    && cd /usr/src/memcached \
    && ./configure \
            --enable-sasl \
    && make -j "$(nproc)" \
    && make install \
    \
    && cd / && rm -rf /usr/src/memcached \
    && apt-mark manual \
        libevent-2.0-5 \
        libsasl2-2 \
    && apt-get purge -y --auto-remove $buildDeps

ENTRYPOINT ["docker-php-entrypoint"]