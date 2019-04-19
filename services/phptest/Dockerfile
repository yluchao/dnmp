FROM ubuntu:16.04
RUN apt-get update \
    && apt-get install \
    python-software-properties \
    software-properties-common \
    && add-apt-repository ppa:ondrej/php \
    # 相关依赖必须手动安装
    && apt-get install -y --no-install-recommends \
        php7.1 \
        php7.1-mysql \
        php7.1-fpm \
        php7.1-json \
        php7.1-mbstring \
        php7.1-xml \
        php7.1-intl \
        php7.1-gd \
        php7.1-opcache \
        php7.1-cgi \
        php7.1-zip \
        php7.1-curl \
        git \
        vim \
        procps \
    && rm -rf /var/lib/apt/lists/*

