FROM php:7.2-fpm

RUN apt-get clean && apt-get update \
    && apt-get install -y locales wget curl software-properties-common ssh build-essential curl git unzip gettext-base \
        libmcrypt-dev libicu-dev libxslt-dev libxml2-dev libglib2.0-bin libcurl4-gnutls-dev libjpeg62-turbo-dev libpng-dev \
        gosu libmagickwand-dev iproute2 libzip-dev \
    # Fix docker run permission problem
    && chmod +s /usr/sbin/gosu \
    # verify that the binary works
    && gosu nobody true \
    && mkdir -p /run/php \
    && rm -rf /var/lib/apt/lists/*

# PHP
COPY config/php_requires.txt /tmp/php_requires

RUN docker-php-ext-install -j$(nproc) `grep -vE "^\s*#" /tmp/php_requires | tr "\n" " "` \
    && pecl install xdebug && docker-php-ext-enable xdebug \
    && pecl install imagick && docker-php-ext-enable imagick \
    && sed -e 's#error_log = .*#error_log = /var/log/php-fpm.log#' -i /usr/local/etc/php-fpm.conf \
    && sed -e 's/listen = .*/listen = \[::\]:9003/' -i /usr/local/etc/php-fpm.d/zz-docker.conf \
    && sed -e 's/;listen\.owner/listen.owner/' -i /usr/local/etc/php-fpm.d/www.conf \
    && sed -e 's/;listen\.group/listen.group/' -i /usr/local/etc/php-fpm.d/www.conf \
    && sed -e 's/;catch_workers_output/catch_workers_output/' -i /usr/local/etc/php-fpm.d/www.conf \
    && sed -e 's/;php_admin_flag[log_errors]/php_admin_flag[log_errors]/' -i /usr/local/etc/php-fpm.d/www.conf \
    && sed -e 's#;php_admin_value[error_log] = .*#php_admin_value[error_log] = /var/log/php/www.log#' -i /usr/local/etc/php-fpm.d/www.conf \
    && curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer \
    && composer global require hirak/prestissimo \
    && rm -rf /var/lib/apt/lists/*

# EZ extra
RUN \
    apt-get update && \
    apt-get install libxslt-dev -y && \
    rm -rf /var/lib/apt/lists/* && \
    docker-php-ext-install exif && \
    docker-php-ext-install xsl

COPY config/custom.php.ini /usr/local/etc/php/conf.d/99-custom.ini.dist
COPY config/log.php.ini /usr/local/etc/php/conf.d/90-log.ini.dist
COPY config/xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini.dist

# XDEBUG!
EXPOSE 9000
# PHP-FPM
EXPOSE 9003

COPY bin/entrypoint.sh /usr/local/bin/entrypoint.sh
COPY bin/wait-for-it.sh /usr/local/bin/wait-for-it.sh
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]

VOLUME /var/www
WORKDIR /var/www
