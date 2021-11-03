FROM joomla:php8.0-fpm
LABEL maintainer="KAINOTOMO PH LTD <info@kainotomo.com>"

ARG DEBIAN_FRONTEND=noninteractive

# Install the PHP extensions
COPY ./supervisord.conf /etc/supervisor/conf.d/
RUN set -ex; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		openssh-server \
		libxml2-dev \
		nginx \
		supervisor \
	; \
docker-php-ext-install \
	soap;

# Cleanup
RUN rm -rf /etc/php/8.0
RUN apt-get clean; rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/* \
    && export TERM=xterm

ENV NGINX_CONF_DIR=/etc/nginx
COPY ./configs/nginx.conf ${NGINX_CONF_DIR}/nginx.conf
COPY ./configs/app.conf ${NGINX_CONF_DIR}/sites-enabled/app.conf
COPY ./configs/www.conf /etc/php/8.0/fpm/pool.d/www.conf

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

EXPOSE 80 443 2222

# Start web server
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
