FROM php:7.2-fpm

LABEL maintainer="Mriyam Tamuli <mbtamuli@gmail.com>"
LABEL org.label-schema.schema-version="1.0.0-rc1"
LABEL org.label-schema.vendor="EasyEngine"
LABEL org.label-schema.name="php"

RUN set -ex; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		imagemagick \
		less \
		libc-client-dev \
		libfreetype6-dev \
		libjpeg-dev \
		libjpeg62-turbo-dev \
		libkrb5-dev \
		libmagickwand-dev \
		libmcrypt-dev \
		libmemcached-dev \
		libxml2-dev \
		libpng-dev \
		libzip-dev \
		mysql-client \
		ssmtp \
		unzip \
		vim \
		zip

RUN pecl install imagick memcached; \
	printf "\n" | pecl install mcrypt-1.0.1; \
	printf "\n" | pecl install redis; \
	docker-php-ext-configure gd --enable-gd-native-ttf --with-freetype-dir=/usr/include/freetype2 --with-png-dir=/usr/include --with-jpeg-dir=/usr/include; \
	docker-php-ext-configure zip --with-libzip; \
	docker-php-ext-configure --with-php-config=/usr/local/bin/php-config; \
	docker-php-ext-configure imap --with-kerberos --with-imap-ssl; \
	docker-php-ext-install gd imap mysqli opcache soap zip; \
	docker-php-ext-enable imagick mcrypt redis; \
	echo "extension=memcached.so" >> /usr/local/etc/php/conf.d/memcached.ini; \
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	rm -rf /var/lib/apt/lists/*;

RUN	mkdir -p /usr/src/php/ext; \
	cd /usr/src/php/ext/; \
	curl -sSL -o php7.zip https://github.com/websupport-sk/pecl-memcache/archive/NON_BLOCKING_IO_php7.zip; \
	unzip php7.zip; \
	mv pecl-memcache-NON_BLOCKING_IO_php7 memcache; \
	rm -rf /tmp/pecl-memcache-php7 /usr/src/php/ext/php7.zip; \
	docker-php-ext-configure memcache --with-php-config=/usr/local/bin/php-config; \
  docker-php-ext-install memcache;

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=2'; \
		echo 'opcache.fast_shutdown=1'; \
		echo 'opcache.enable_cli=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini

# Donwload and install composer
RUN curl -sSL "https://getcomposer.org/installer" | php \
		&& mv composer.phar /usr/local/bin/composer

# Install wp-cli
RUN curl -O "https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar" \
		&& chmod +x wp-cli.phar \
		&& mv wp-cli.phar /usr/local/bin/wp

# Setup a config file
RUN mkdir -p /etc/wp-cli
RUN { \
		echo 'path: /var/www/htdocs'; \
		} > /etc/wp-cli/config.yml

RUN echo "mailhub=postfix\nUseTLS=NO\nFromLineOverride=YES" > /etc/ssmtp/ssmtp.conf

RUN curl -L https://download.newrelic.com/php_agent/release/newrelic-php5-8.6.0.238-linux.tar.gz | tar -C /tmp -zx && \
export NR_INSTALL_USE_CP_NOT_LN=1 && \
export NR_INSTALL_SILENT=1 && \
/tmp/newrelic-php5-*/newrelic-install install && \
rm -rf /tmp/newrelic-php5-* /tmp/nrinstall*

ENV NR_PORT=/run/newrelic/newrelic.sock

COPY expose_off.ini /usr/local/etc/php/conf.d/expose_off.ini
COPY bashrc /root/.bashrc
COPY bashrc /var/www/.bashrc
COPY docker-entrypoint.sh /usr/local/bin/
COPY newrelic.ini /usr/local/etc/php/conf.d/newrelic.ini

RUN mkdir -p /var/log/php /var/log/newrelic /run/newrelic; \
	chown -R www-data:www-data /var/log/php; \
	chown -R www-data:www-data /var/log/newrelic; \
	chown -R www-data:www-data /usr/local/etc/php/conf.d; \
	rm /usr/local/etc/php-fpm.d/*;
COPY php.ini /usr/local/etc/php/php.ini
COPY easyengine.conf /usr/local/etc/php-fpm.d/easyengine.conf

WORKDIR /var/www/htdocs
USER www-data

ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["php-fpm"]
