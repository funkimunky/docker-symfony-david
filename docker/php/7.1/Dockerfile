FROM php:7.1-apache

RUN a2enmod rewrite
RUN service apache2 restart

RUN apt-get update && apt-get install -y curl

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
        libpng12-dev \
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

# Copy over Xdebug Configuration.
COPY /xdebug.ini /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# Copy over php.ini settings
COPY /php.ini /usr/local/etc/php/php.ini

# Setup the Composer installer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

#RUN php -r "if (hash_file('SHA384', 'composer-setup.php') === '544e09ee996cdf60ece3804abc52599c22b1f40f4323403c44d44fdfdd586475ca9813a858088ffbc1f233e9b180f061') { echo 'Installer verified'; } else { #echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

RUN php composer-setup.php --install-dir=/usr/bin --filename=composer

RUN php -r "unlink('composer-setup.php');"

RUN chmod +x /usr/bin/composer 

COPY 001-symfony.local.conf /etc/apache2/sites-available/001-symfony.local.conf

RUN a2dissite 000-default.conf

RUN a2ensite 001-symfony.local.conf

RUN service apache2 restart

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

WORKDIR /var/www/html/symfony

COPY /supervisord.conf /etc/supervisor/supervisord.conf
COPY /.bashrc /root/.bashrc

# Configure cron.
RUN rm -Rf /etc/cron.daily  && \
    rm -Rf /etc/cron.weekly && \
    rm -Rf /etc/cron.monthly && \
    rm -Rf /etc/cron.hourly

CMD exec supervisord -n