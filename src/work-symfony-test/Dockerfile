FROM php:7.1-apache

# Setup of time zone
ENV TZ=Europe/Moscow

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
	echo $TZ > /etc/timezone && \
	echo "date.timezone = $TZ" > $PHP_INI_DIR/conf.d/timezone.ini


# Setup of Linux libs
RUN apt update && apt install -y \
	apt-transport-https \
	wget \
	git \
	libfreetype6-dev \
	libjpeg62-turbo-dev \
	libmcrypt-dev \
	libpng-dev \
	libpq-dev \
	zlib1g-dev \
	vim \
	locate

# Add additional repos
RUN wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg \
	&& echo "deb https://packages.sury.org/php/ jessie main" > /etc/apt/sources.list.d/php.list \
	&& apt update \
	&& apt purge 'php.*'

# Setup of PHP extensions
RUN apt install -y \
	php-common \
	php7.1-common \
	php7.1-odbc \
	php7.1-mysql \
	php7.1-sqlite3 \
	php7.1-iconv \
	php7.1-mcrypt \
	php7.1-mbstring \
	php7.1-redis \
	php7.1-json \
	php7.1-imagick \
	php7.1-curl

RUN pecl install -o -f xdebug && \
	rm -rf /tmp/pear

# Installing ICU version 60.1
RUN curl -sS -o /tmp/icu.tar.gz -L http://download.icu-project.org/files/icu4c/60.1/icu4c-60_1-src.tgz && tar -zxf /tmp/icu.tar.gz -C /tmp && cd /tmp/icu/source && ./configure --prefix=/usr/local && make && make install

# PHP_CPPFLAGS are used by the docker-php-ext-* scripts
ENV PHP_CPPFLAGS="$PHP_CPPFLAGS -std=c++11"

RUN docker-php-ext-configure intl \
	--with-icu-dir=/usr/local && \
    docker-php-ext-configure pgsql \
	-with-pgsql=/usr/local/pgsql && \
    docker-php-ext-configure gd \
	--with-freetype-dir=/usr/include/ \
        --with-png-dir=/usr/include/ \
        --with-jpeg-dir=/usr/include/ && \
    docker-php-ext-install \
	opcache \
	intl \
	pdo_pgsql \
	pdo_mysql \
	gd \
	zip

# Allow superuser to use composer
ENV COMPOSER_ALLOW_SUPERUSER 1

# Setup of composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
    php -r "if (hash_file('SHA384', 'composer-setup.php') === '544e09ee996cdf60ece3804abc52599c22b1f40f4323403c44d44fdfdd586475ca9813a858088ffbc1f233e9b180f061') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" && \
    php composer-setup.php && \
    php -r "unlink('composer-setup.php');" && \
    mv composer.phar /usr/local/bin/composer && \
    chmod +x /usr/local/bin/composer

RUN composer global require "fxp/composer-asset-plugin:~1.3"

# Initializing Apache2 modules
RUN a2enmod proxy && \
    a2enmod proxy_http && \
    a2enmod rewrite

# Install Symfony
RUN mkdir -p /usr/local/bin
RUN curl -LsS https://symfony.com/installer -o /usr/local/bin/symfony
RUN chmod a+x /usr/local/bin/symfony

# Setup of cyrillic fonts
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y locales

RUN sed -i -e 's/# ru_RU.UTF-8 UTF-8/ru_RU.UTF-8 UTF-8/' /etc/locale.gen && \
    dpkg-reconfigure --frontend=noninteractive locales && \
    update-locale LANG=ru_RU.UTF-8

ENV LANG ru_RU.UTF-8

# Setup of virtual host
# Setup your VIRTUAL_HOST, VIRTUAL_HOST_ALIAS and DOC_ROOT as env variables in docker-compose.yml
# The rest will be taken care of by 000-default.conf - Apache configuration file
COPY src/apache2/000-default.conf /etc/apache2/sites-available

# Setup of bash environment
COPY src/bash/.bashrc /root/.bashrc
COPY src/bash/.vimrc /root/.vimrc
COPY src/bash/.vim /root/.vim
COPY src/bash/.inputrc /root/.inputrc

# Setup of php.ini
COPY src/php/7.1/php.ini /usr/local/etc/php/php.ini

# Add main start script for when image launches
# COPY run.sh /run.sh
# RUN chmod 0755 /run.sh

WORKDIR /var/www/html

# CMD ["/run.sh"]
