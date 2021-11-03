FROM joomla
LABEL maintainer="KAINOTOMO PH LTD <info@kainotomo.com>"

# Install the PHP extensions
RUN set -ex; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		openssh-server \
		libxml2-dev \
	; \
docker-php-ext-install \
	soap;

VOLUME /var/www/html

#INSTALL APCU
#RUN echo "extension=apcu.so" >> /usr/local/etc/php/conf.d/docker-php-ext-apcu.ini
RUN echo "apc.enable_cli=1" >> /usr/local/etc/php/conf.d/docker-php-ext-apcu.ini
RUN echo "apc.shm_size=128M" >> /usr/local/etc/php/conf.d/docker-php-ext-apcu.ini
#RUN echo "apc.enable=1" >> /usr/local/etc/php/conf.d/docker-php-ext-apcu.ini

# Install OpenSSH and set the password for root to "Docker!". In this example, "apk add" is the install instruction for an Alpine Linux-based image.
COPY sshd_config /etc/ssh/
RUN echo "root:Docker!" | chpasswd 

COPY entrypoint.sh /root/entrypoint.sh
ENTRYPOINT ["/root/entrypoint.sh"]

# Start web server
CMD ["apache2-foreground"]

EXPOSE 80 2222
