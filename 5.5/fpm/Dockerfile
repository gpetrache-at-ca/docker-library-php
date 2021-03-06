FROM centos:6.8

# Add Repos - EPEL, Remi && Update & Upgrade
USER root
RUN rpm -Uvh http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm \
    && rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm \
    && yum repolist \
    && yum update -y \
    && yum upgrade -y

# Install "PHP", "PHP Extentions", "libraries", "Software's"
USER root
RUN yum --enablerepo remi-php55,remi,epel install -y \
        php \
        php-cli \
        php-fpm \
        php-common \
        \       
        php-bcmath \
        php-dba \
        php-devel \
        php-enchant \
        php-gd \
        php-imap \
        php-intl \
        php-ldap \
        php-mbstring \
        php-mcrypt \
        php-mysqlnd \
        php-oauth \
        php-odbc \
        php-opcache \
        php-pdo \
        php-pear \
        php-pgsql \
        php-process \
        php-pspell \
        php-snmp \
        php-soap \
        php-xml \
        php-xmlrpc \
        \
        php-pecl-amqp \       
        php-pecl-igbinary \
        php-pecl-imagick \
        php-pecl-jsonc \
        php-pecl-jsonc-devel \
        php-pecl-memcache \
        php-pecl-mongo \
        php-pecl-oauth \
        php-pecl-redis \
        php-pecl-ssh2 \
        php-pecl-zip \
    # Update PECL channels       
    && pecl update-channels \
    # Clean Up
    && yum clean metadata \
    && yum clean all -y \    
    && rm -rf \
        /tmp/* \
        /var/tmp/*

RUN sed -i 's/daemonize = yes/daemonize = no/' /etc/php-fpm.conf \
    && sed -i 's/listen.allowed_clients = 127.0.0.1/;listen.allowed_clients = 127.0.0.1/' /etc/php-fpm.d/www.conf \
    && sed -i 's/listen = 127.0.0.1:9000/listen = [::]:9000/' /etc/php-fpm.d/www.conf \
    && sed -i 's/;catch_workers_output = yes/catch_workers_output = yes/' /etc/php-fpm.d/www.conf \
    && { \
        echo '[www]'; \
#        echo 'access.log = /dev/stdout'; \
        echo 'clear_env = no'; \
    } | tee /etc/php-fpm.d/docker.conf

USER root
COPY docker-php-entrypoint /usr/local/bin/

ENTRYPOINT ["docker-php-entrypoint"]

WORKDIR /var/www/html

EXPOSE 9000
CMD ["php-fpm", "-F"]
