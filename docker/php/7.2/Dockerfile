FROM php:7.2-apache

RUN a2enmod rewrite
RUN a2enmod ssl
RUN mkdir /etc/apache2/ssl
WORKDIR /etc/apache2/ssl
RUN openssl genrsa -out ca.key 2048
RUN openssl req -nodes -new -key ca.key -out ca.csr -subj "/C=UK/ST=Stirlingshire/L=Stirling/O=OrgName/OU=IT Department/CN=localhost"
RUN openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt
COPY /000-default.conf /etc/apache2/sites-available/000-default.conf
RUN service apache2 restart

RUN apt-get update && apt-get install -my curl wget gnupg

# Install nodejs
WORKDIR /root
RUN curl -sL https://deb.nodesource.com/setup_8.x -o nodesource_setup.sh
RUN bash nodesource_setup.sh
RUN apt-get update && apt-get install -y nodejs

# Install yarn
RUN curl -o- -L https://yarnpkg.com/install.sh | bash

# Install PHP 7.1 Extensions
RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng-dev \
        libssl-dev \
        libcurl3-dev \
        libxml2-dev \
        libzzip-dev \
        libldap2-dev  \
        libicu-dev \
        libxslt-dev \
        libc-client-dev \
        libkrb5-dev \
        cron \
        supervisor \
        nano \
        xterm \
    && docker-php-ext-install mbstring xml zip soap mysqli \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-install gd imap phar pdo_mysql \
    && pecl install xdebug \
    && docker-php-ext-enable xdebug

# Copy over php.ini settings
COPY /php.ini /usr/local/etc/php/php.ini

# Setup the Composer installer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php composer-setup.php --install-dir=/usr/bin --filename=composer \
    && php -r "unlink('composer-setup.php');" \
    && chmod +x /usr/bin/composer 

# setup vhosts

RUN a2dissite 000-default.conf

COPY 001-symfony.local.conf /etc/apache2/sites-available/001-symfony.local.conf
RUN a2ensite 001-symfony.local.conf

COPY 002-the_spacebar.local.conf /etc/apache2/sites-available/002-the_spacebar.local.conf
RUN a2ensite 002-the_spacebar.local.conf

COPY 003-robocode_co_uk.local.conf /etc/apache2/sites-available/003-robocode_co_uk.local.conf
RUN a2ensite 003-robocode_co_uk.local.conf

COPY 004-10thperthshire.local.conf /etc/apache2/sites-available/004-10thperthshire.local.conf
RUN a2ensite 004-10thperthshire.local.conf

# setup vhosts end

# install sshd
RUN apt-get update && apt-get install -y openssh-server
RUN mkdir /var/run/sshd
RUN echo 'root:screencast' | chpasswd
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile
RUN mkdir /root/.ssh
COPY ./id_rsa.pub /tmp/id_rsa.pub
RUN cat /tmp/id_rsa.pub >> /root/.ssh/authorized_keys && rm -f /tmp/id_rsa.pub
EXPOSE 22

WORKDIR /var/www/html

COPY /supervisord.conf /etc/supervisor/supervisord.conf
COPY /.bashrc /root/.bashrc

# Configure cron.
RUN rm -Rf /etc/cron.daily  && \
    rm -Rf /etc/cron.weekly && \
    rm -Rf /etc/cron.monthly && \
    rm -Rf /etc/cron.hourly

#install git
RUN apt-get update && apt-get install -y git-all

# Copy over Xdebug Configuration.
COPY /xdebug.ini /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN service apache2 restart

WORKDIR /usr/bin
RUN wget -O phpunit https://phar.phpunit.de/phpunit-7.phar \
&& chmod +x phpunit

WORKDIR /var/www/html
CMD exec supervisord -n
