FROM ubuntu:22.04 AS base_image
LABEL maintainer="vuongnq"

#### PHP-FPM + NGINX
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update --fix-missing -y
RUN apt-get upgrade -y
RUN yes | apt-get install nginx
RUN yes | apt-get install software-properties-common && add-apt-repository ppa:ondrej/php
RUN apt-get clean 
RUN apt-get install -y \
    vim \
    cron \
    bind9 \
    git \
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
    g++ \
    build-essential \
    libaio1 \
    php8.1-fpm \
    php8.1-cli \
    php8.1-curl \
    php8.1-mysql \
    php8.1-mbstring \
    php8.1-xml \
    php8.1-dev \
    php8.1-mongodb \
    php8.1-gd \
    php8.1-intl \
    php8.1-xsl \
    php8.1-zip \
    php8.1-mbstring \
    php-pear 
RUN php -r "readfile('http://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer && mkdir /run/php 
RUN echo "clear_env = no">>/etc/php/8.1/fpm/pool.d/www.conf
RUN echo "catch_workers_output = yes">>/etc/php/8.1/fpm/pool.d/www.conf
RUN sed -i 's/\/var\/log\/php8.1-fpm.log/\/proc\/self\/fd\/2/g' /etc/php/8.1/fpm/php-fpm.conf
RUN sed -i -E 's/short_open_tag =.*/short_open_tag = On/' /etc/php/8.1/fpm/php.ini
RUN sed -i -E 's/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/' /etc/php/8.1/fpm/php.ini
RUN sed -i -E 's/upload_max_filesize = 2M/upload_max_filesize = 200M/' /etc/php/8.1/fpm/php.ini
RUN sed -i -E 's/post_max_size = 8M/post_max_size = 50M/' /etc/php/8.1/fpm/php.ini
RUN sed -i -E 's/memory_limit = 128M/memory_limit = 256M/' /etc/php/8.1/fpm/php.ini
RUN sed -i -E 's/max_input_time = 60/max_input_time = 3600/' /etc/php/8.1/fpm/php.ini
RUN sed -i -E 's/max_execution_time = 30/max_execution_time = 3600/' /etc/php/8.1/fpm/php.ini

ENV INSTANT_CLIENT=/usr/lib/oracle/instantclient_21_8
RUN mkdir -p /usr/lib/oracle

RUN cd /opt \
    && PHP_VERSION=$(php -v | grep -Po '(?<=PHP\s)[\d.]+') \
    && BRANCH="PHP-${PHP_VERSION}" \
    && git clone https://github.com/php/php-src.git \
    && cd php-src \
    && git checkout ${BRANCH}

RUN cd /opt \
    && curl https://download.oracle.com/otn_software/linux/instantclient/218000/instantclient-basic-linux.x64-21.8.0.0.0dbru.zip -O \
    && curl https://download.oracle.com/otn_software/linux/instantclient/218000/instantclient-sdk-linux.x64-21.8.0.0.0dbru.zip -O 

RUN cd /opt \
    && yes | unzip instantclient-basic-linux.x64-21.8.0.0.0dbru.zip \
    && yes | unzip instantclient-sdk-linux.x64-21.8.0.0.0dbru.zip \
    && mv instantclient_21_8 ${INSTANT_CLIENT}

RUN echo ${INSTANT_CLIENT} > /etc/ld.so.conf.d/oracle.conf
RUN ldconfig

RUN ls -al /opt
RUN ls /usr/lib/oracle/instantclient_21_8
RUN cd /opt/php-src/ext/oci8 \
    && phpize \
    && ./configure --with-php-config=/usr/bin/php-config --with-oci8=instantclient,${INSTANT_CLIENT} \
    && make \
    && make install

RUN cd /opt/php-src/ext/pdo_oci \
    && phpize \
    &&./configure --with-php-config=/usr/bin/php-config --with-pdo-oci=instantclient,${INSTANT_CLIENT} \
    && make \
    && make install

RUN echo "extension=pdo_oci.so" > /etc/php/8.1/fpm/conf.d/10-pdo_oci.ini
RUN echo "extension=oci8.so" > /etc/php/8.1/fpm/conf.d/10-oci8.ini
RUN echo "extension=pdo_oci.so" > /etc/php/8.1/cli/conf.d/10-pdo_oci.ini
RUN echo "extension=oci8.so" > /etc/php/8.1/cli/conf.d/10-oci8.ini

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

