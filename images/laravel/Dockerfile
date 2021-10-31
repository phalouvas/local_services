FROM php:8.0-apache
LABEL maintainer="KAINOTOMO PH LTD <info@kainotomo.com>"

# Enable Apache Rewrite Module
RUN a2enmod rewrite

# apache configs + document root
ENV APACHE_DOCUMENT_ROOT=/var/www/html/public
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

# Install the PHP extensions
RUN set -ex; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	\
	apt-get update; \
	apt-get install -y --no-install-recommends \
		libbz2-dev \
		libgmp-dev \
		libjpeg-dev \
		libldap2-dev \
		libmcrypt-dev \
		libpng-dev \
		libpq-dev \
		libzip-dev \
		libxml2-dev \
		supervisor \
		cron \
		openssl \
		git \
		openssh-server \
		nodejs \
		npm \
	; \
	\
	docker-php-ext-configure gd --with-jpeg; \
	debMultiarch="$(dpkg-architecture --query DEB_BUILD_MULTIARCH)"; \
	docker-php-ext-configure ldap --with-libdir="lib/$debMultiarch"; \
	docker-php-ext-install -j "$(nproc)" \
		bz2 \
		gd \
		gmp \
		ldap \
		mysqli \
		pdo_mysql \
		zip \
		soap \
	; \
	\
# pecl will claim success even if one install fails, so we need to perform each install separately
	pecl install APCu-5.1.20; \
	pecl install redis-5.3.4; \
	\
	docker-php-ext-enable \
		apcu \
		redis \
	; \
	rm -r /tmp/pear;
	#\
# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
	#apt-mark auto '.*' > /dev/null; \
	#apt-mark manual $savedAptMark; \
	#ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
	#	| awk '/=>/ { print $3 }' \
	#	| sort -u \
	#	| xargs -r dpkg-query -S \
	#	| cut -d: -f1 \
	#	| sort -u \
	#	| xargs -rt apt-mark manual; \
	#\
	#apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	#rm -rf /var/lib/apt/lists/*

VOLUME /var/www/html

#INSTALL APCU
#RUN echo "extension=apcu.so" >> /usr/local/etc/php/conf.d/docker-php-ext-apcu.ini
RUN echo "apc.enable_cli=1" >> /usr/local/etc/php/conf.d/docker-php-ext-apcu.ini
RUN echo "apc.shm_size=128M" >> /usr/local/etc/php/conf.d/docker-php-ext-apcu.ini
#RUN echo "apc.enable=1" >> /usr/local/etc/php/conf.d/docker-php-ext-apcu.ini

# Composer.
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Install OpenSSH and set the password for root to "Docker!". In this example, "apk add" is the install instruction for an Alpine Linux-based image.
COPY sshd_config /etc/ssh/
RUN echo "root:Docker!" | chpasswd 

COPY entrypoint.sh /home/entrypoint.sh
ENTRYPOINT ["/home/entrypoint.sh"]

# Start web server
#CMD ["/usr/bin/supervisord"]
CMD ["apache2-foreground"]

EXPOSE 80 2222
