FROM ubuntu:20.04 AS base_image
LABEL maintainer="Tuha"

#### PHP-FPM + NGINX
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update
RUN yes | apt-get install nginx
RUN yes | apt-get install software-properties-common && add-apt-repository ppa:ondrej/php
RUN apt-get clean 
RUN apt-get update --fix-missing -y
RUN apt-get upgrade -y
RUN apt-get install -y vim cron
RUN apt-get install bind9 -y
RUN apt-get update -y
RUN apt-get upgrade -y
RUN apt-get install -y \
    php8.1-fpm \
    php8.1-cli \
    php8.1-curl \
    php8.1-mysql \
    php8.1-mbstring \
    php8.1-xml \
    php8.1-dev \
    php8.1-redis \
    php8.1-mongodb \
    php8.1-gd \
    php8.1-intl \
    php8.1-xsl \
    php8.1-zip \
    php8.1-mbstring 
RUN php -r "readfile('http://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer && mkdir /run/php 
RUN echo "clear_env = no">>/etc/php/8.1/fpm/pool.d/www.conf
RUN echo "catch_workers_output = yes">>/etc/php/8.1/fpm/pool.d/www.conf
RUN sed -i 's/\/var\/log\/php8.1-fpm.log/\/proc\/self\/fd\/2/g' /etc/php/8.1/fpm/php-fpm.conf
RUN sed -i -E 's/session\.save_handler =.*/session\.save_handler = redis/' /etc/php/8.1/fpm/php.ini
RUN sed -i -E 's/;session\.save_path =.*/session\.save_path = "tcp:\/\/redis"/' /etc/php/8.1/fpm/php.ini
RUN sed -i -E 's/short_open_tag =.*/short_open_tag = On/' /etc/php/8.1/fpm/php.ini
RUN sed -i -E 's/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/' /etc/php/8.1/fpm/php.ini
RUN sed -i -E 's/upload_max_filesize = 2M/upload_max_filesize = 50M/' /etc/php/8.1/fpm/php.ini
RUN sed -i -E 's/post_max_size = 8M/post_max_size = 50M/' /etc/php/8.1/fpm/php.ini
RUN sed -i -E 's/memory_limit = 128M/memory_limit = 256M/' /etc/php/8.1/fpm/php.ini
RUN sed -i -E 's/max_input_time = 60/max_input_time = 180/' /etc/php/8.1/fpm/php.ini
RUN sed -i -E 's/max_execution_time = 30/max_execution_time = 180/' /etc/php/8.1/fpm/php.ini

RUN apt-get install -y \
    supervisor \
    zip \
    curl \
    sudo \
    unzip \
    libicu-dev \
    libbz2-dev \
    libpng-dev \
    libjpeg-dev \
    libmcrypt-dev \
    libreadline-dev \
    libfreetype6-dev \
    g++

#### Clear apt-get
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN echo '* * * * * cd /var/www; /usr/bin/php -f cron.php > /dev/null 2>&1' > /etc/cron.d/espo-cron
RUN chmod 0644 /etc/cron.d/espo-cron
RUN crontab /etc/cron.d/espo-cron
RUN touch /var/log/cron.log


#### RUNING
EXPOSE 80
COPY ./supervisord/supervisord.conf  /etc/supervisor/conf.d/supervisord.conf
COPY ./initial /usr/local/bin/
RUN chmod +x /usr/local/bin/initial

RUN mkdir -p /var/log/initial
RUN touch /var/log/initial/log
RUN chmod -R 775 /var/log/initial/log
CMD [ "/usr/local/bin/initial"]

