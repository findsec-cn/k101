FROM php:7.2-fpm
MAINTAINER shidongliang<shidongliang@xin.com>

# Change Timezone
ENV TIME_ZONE Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TIME_ZONE /etc/localtime && echo $TIME_ZONE > /etc/timezone
# Change APT SOURCES
COPY sources.list /etc/apt/sources.list
RUN rm /etc/apt/sources.list.d/buster.list

RUN apt-get update && apt-get install -y --no-install-recommends \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng-dev \
        libmemcached-dev \
        graphicsmagick \
        libgraphicsmagick1-dev \
        imagemagick \
        libmagickwand-dev \
        libssh2-1-dev \
        libzip-dev \
        libzookeeper-mt-dev \
        libldap2-dev \
        libssl-dev \
        libmosquitto-dev \
        librabbitmq-dev \
        inotify-tools \
        libevent-dev \
        libhiredis-dev \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ \
    && docker-php-ext-install -j$(nproc) bcmath gd gettext pdo_mysql mysqli ldap pcntl soap sockets sysvsem xmlrpc \
    && pecl install gmagick-2.0.5RC1 \
    && pecl install imagick-3.4.3 \
    && pecl install memcached-3.0.4 \
    && pecl install redis-4.0.2 \
    && pecl install mcrypt-1.0.1 \
    && pecl install mongodb-1.4.3 \
    && pecl install swoole-2.1.3 \
    && pecl install ssh2-1.1.2 \
    && pecl install yaf-3.0.7 \
    && pecl install yaconf-1.0.7 \
    && pecl install zip-1.15.2 \
    && pecl install zookeeper-0.5.0 \
    && pecl install Mosquitto-0.4.0 \
    && pecl install amqp-1.9.3 \
    && pecl install inotify-2.0.0 \
    && pecl install event-2.3.0 \
    && pecl install xdebug-2.6.0 \
    && docker-php-ext-enable gmagick memcached redis mcrypt mongodb swoole ssh2 yaf yaconf zip zookeeper mosquitto amqp inotify event \
    && apt-get clean \
    && apt-get autoremove --purge -y

# Install php composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php -r "if (hash_file('sha384', 'composer-setup.php') === '93b54496392c062774670ac18b134c3b3a95e5a5e5c8f1a9f115f203b75bf9a129d5daa8ba6a13e2cc8a1da0806388a8') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" \
    && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
    && chmod +x /usr/local/bin/composer \
    && php -r "unlink('composer-setup.php');" \
    && composer config -g repo.packagist composer https://packagist.laravel-china.org

# Make dir
RUN mkdir -p /data/logs/php /data/cache/upload_tmp
COPY php-fpm.conf /usr/local/etc/php-fpm.conf
COPY php.ini /usr/local/etc/php/php.ini

