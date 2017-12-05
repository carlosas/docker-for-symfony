FROM php:7.1-fpm
ARG TIMEZONE

RUN apt-get update && apt-get install -y git unzip openssl

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN composer --version

# Set timezone
RUN ln -snf /usr/share/zoneinfo/${TIMEZONE} /etc/localtime && echo ${TIMEZONE} > /etc/timezone
RUN printf '[PHP]\ndate.timezone = "%s"\n', ${TIMEZONE} > /usr/local/etc/php/conf.d/tzone.ini

# Install PDO extensions
RUN docker-php-ext-install pdo pdo_mysql

# Install Redis extension
RUN pecl install -o -f redis && rm -rf /tmp/pear && docker-php-ext-enable redis

# Install xdebug
RUN pecl install xdebug
RUN docker-php-ext-enable xdebug
RUN echo "error_reporting = E_ALL" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "display_startup_errors = On" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "display_errors = On" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_enable=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_connect_back=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.idekey=\"PHPSTORM\"" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_port=9001" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# Install PHPUnit
RUN composer global require "phpunit/phpunit"
RUN ln -s /root/.composer/vendor/bin/phpunit /usr/bin/phpunit

# Create an alias for Symfony3 console
RUN echo 'alias symfony="php bin/console"' >> ~/.bashrc

# Prevent cache/logs/sessions permission problems
#RUN HTTPDUSER=$(ps axo user,comm | grep -E '[a]pache|[h]ttpd|[_]www|[w]ww-data|[n]ginx' | grep -v root | head -1 | cut -d\  -f1) && setfacl -dR -m u:"$HTTPDUSER":rwX -m u:$(whoami):rwX /var/www/symfony/var && setfacl -R -m u:"$HTTPDUSER":rwX -m u:$(whoami):rwX /var/www/symfony/var

WORKDIR /var/www/symfony
