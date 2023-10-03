FROM phpdockerio/php:7.4-fpm
WORKDIR "/application"

RUN apt-get update \
    && apt-get -y --no-install-recommends install \
        php7.4-bcmath \ 
        php7.4-imagick \ 
        php7.4-intl \ 
        php7.4-memcache \ 
        php7.4-memcached \ 
        php7.4-mysql \ 
        php7.4-redis \ 
        php7.4-xdebug \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/*
