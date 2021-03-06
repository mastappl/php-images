FROM mastappl/php-fpm:7.2-base

MAINTAINER d2cio <reg@d2c.io>

# Set ENVIROINMENTS
ENV MAGENTO_VERSION 2.3.0
ENV INSTALL_DIR /var/www/html/magento2-$MAGENTO_VERSION
ENV COMPOSER_HOME /root/.composer

# Install extensions
RUN apt-get update && \
    apt-mark showmanual > /tmp/savedAptMark && \
    docker-php-source extract && \
    apt-get install -y libicu-dev libxml2-dev libxslt-dev libwebp-dev libfreetype6-dev libgif-dev libpng-dev libjpeg-dev libjpeg62-turbo-dev --no-install-recommends && \
    docker-php-ext-configure gd --with-webp-dir=/usr --with-png-dir=/usr --with-jpeg-dir=/usr --with-freetype-dir=/usr && \
    docker-php-ext-install mysqli opcache intl soap bcmath pdo_mysql xsl gd zip && \
# Install Redis Pecl package
    pecl install -f redis && \
    docker-php-ext-enable redis && \
# Cleaner
    docker-php-source delete && \
    apt-mark auto '.*' > /dev/null && \
    [ -z "$(cat /tmp/savedAptMark)" ] || apt-mark manual $(cat /tmp/savedAptMark) && \
    find /usr/local -type f -name '*.so*' -exec ldd '{}' ';' | awk '/=>/ { print $(NF-1) }' | sort -u | \
    xargs -r dpkg-query --search | cut -d: -f1 | sort -u | xargs -r apt-mark manual && \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false && \
    rm -rf /tmp/* ~/.pearrc /var/lib/apt/lists/* && \
# Get magento
    curl https://codeload.github.com/magento/magento2/tar.gz/$MAGENTO_VERSION -o $MAGENTO_VERSION.tar.gz && \
    tar xf $MAGENTO_VERSION.tar.gz && \
# Run Composer
    cd $INSTALL_DIR && \
    composer install && \
    mkdir $INSTALL_DIR/var/page_cache && \
    mkdir -p $INSTALL_DIR/pub/media/catalog/product/i && \
    mkdir $INSTALL_DIR/pub/media/catalog/product/cache && \
    chown -R www-data:www-data /var/www && \
    find $INSTALL_DIR -type f -exec chmod 644 {} \; && \
    find $INSTALL_DIR -type d -exec chmod 755 {} \; && \
    find $INSTALL_DIR/var -type d -exec chmod 777 {} \; && \
    find $INSTALL_DIR/pub/media -type d -exec chmod 777 {} \; && \
    find $INSTALL_DIR/pub/static -type d -exec chmod 777 {} \; && \
    chmod 777 $INSTALL_DIR/app/etc && \
    chmod 644 $INSTALL_DIR/app/etc/*.xml && \
    chmod +x $INSTALL_DIR/bin/magento && \
    tar -cJf /var/www/magento_2.tar * && \
    rm -rf /var/www/html

WORKDIR /var/www
