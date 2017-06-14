FROM centos:6.8

ARG TINI_VERSION="0.14.0"
ARG SU_EXEC_VERSION="0.2"
ARG NODE_VERSION=stable
ARG YARN_VERSION=latest
ARG HOST_USERNAME=docker
ARG PUID=1000
ARG PGID=1000

# Add Repos - EPEL, Remi && Update & Upgrade
USER root
RUN rpm -Uvh http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm \
    && rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm \
    && echo -e "[mongodb-org-3.4]\nname=MongoDB Repository\nbaseurl=https://repo.mongodb.org/yum/redhat/\$releasever/mongodb-org/3.4/x86_64/\ngpgcheck=1\nenabled=1\ngpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc" > /etc/yum.repos.d/mongodb-org-3.4.repo \
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
        \
        php-pecl-xdebug \
        \
        # Development tools
        screen \
        tmux \
        vim \
        ack-grep \
        gcc \
        git \
        postgresql \
        mongodb-org-shell \
        mongodb-org-tools \
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

##################
# tini           #
##################
ENV TINI_VERSION ${TINI_VERSION}

RUN curl -fL -o /sbin/tini https://github.com/krallin/tini/releases/download/v${TINI_VERSION}/tini \
  && chmod +x /sbin/tini

##################
# su-exec        #
##################
ENV SU_EXEC_VERSION ${SU_EXEC_VERSION}
ENV SU_EXEC_DOWNLOAD_URL "https://github.com/ncopa/su-exec/archive/v${SU_EXEC_VERSION}.tar.gz"

RUN curl -fL ${SU_EXEC_DOWNLOAD_URL} | tar xfz - \
    && cd su-exec-${SU_EXEC_VERSION} \
    && make \
    && mv su-exec /usr/local/bin \
    && cd / \
    && rm -rf su-exec-${SU_EXEC_VERSION}

##################
# Non-Root User: #
##################
USER root
# Add a non-root user to prevent files being created with root permissions on host machine.
ENV HOST_USERNAME ${HOST_USERNAME}
ENV PUID ${PUID}
ENV PGID ${PGID}

RUN groupadd -g ${PGID} ${HOST_USERNAME} \
    && useradd -u ${PUID} -g ${HOST_USERNAME} -G wheel,apache -m ${HOST_USERNAME}

#####################################
# Composer:
#####################################
USER root
# Setup the Composer installer
RUN curl -o /tmp/composer-setup.php https://getcomposer.org/installer \
    && curl -o /tmp/composer-setup.sig https://composer.github.io/installer.sig \
    && php -r "if (hash('SHA384', file_get_contents('/tmp/composer-setup.php')) !== trim(file_get_contents('/tmp/composer-setup.sig'))) { unlink('/tmp/composer-setup.php'); echo 'Invalid installer' . PHP_EOL; exit(1); }" \
    # Install Composer
    && php /tmp/composer-setup.php --no-ansi --install-dir=/usr/local/bin --filename=composer \
    && rm -rf /tmp/composer-setup.php \
    && composer self-update

# Add the composer.json to host user
RUN mkdir -p /home/${HOST_USERNAME}/.composer/vendor/bin \
    # Add composer vendor bin to PATH
    && echo "" >> /home/${HOST_USERNAME}/.bashrc \
    && echo "export PATH=~/.composer/vendor/bin:$PATH" >> /home/${HOST_USERNAME}/.bashrc

COPY ./composer.json /home/${HOST_USERNAME}/.composer
# Make sure that ~/.composer belongs to host user
RUN chown -R ${HOST_USERNAME}:${HOST_USERNAME} /home/${HOST_USERNAME}/.composer

# Run the global install to host user
USER ${HOST_USERNAME}
RUN composer global install \
    # Install composer development tools
    && composer global require phpunit/phpunit \
    && composer global require phpunit/dbunit

#####################################
# Node / NVM:
#####################################
ENV NODE_VERSION ${NODE_VERSION}
ENV NVM_DIR /home/${HOST_USERNAME}/.nvm

# Install nvm (A Node Version Manager)
RUN curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.1/install.sh | /bin/bash \
    && . $NVM_DIR/nvm.sh \
    && nvm install ${NODE_VERSION} \
    && nvm use ${NODE_VERSION} \
    && nvm alias ${NODE_VERSION}

# Wouldn't execute when added to the RUN statement in the above block
# Source NVM when loading bash since ~/.profile isn't loaded on non-login shell
RUN echo "" >> ~/.bashrc \
    && echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc \
    && echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc

# Add NVM binaries to root's .bashrc
USER root
RUN echo "" >> ~/.bashrc \
    && echo 'export NVM_DIR="/home/${HOST_USERNAME}/.nvm"' >> ~/.bashrc \
    && echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc

#####################################
# YARN:
#####################################
USER ${HOST_USERNAME}
ENV YARN_VERSION ${YARN_VERSION}

RUN [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" \
    && \
    if [ ${YARN_VERSION} = "latest" ]; then \
        curl -o- -L https://yarnpkg.com/install.sh | bash; \
    else \
        curl -o- -L https://yarnpkg.com/install.sh | bash -s -- --version ${YARN_VERSION}; \
    fi \
    && echo "" >> ~/.bashrc \
    && echo 'export PATH="$HOME/.yarn/bin:$PATH"' >> ~/.bashrc


# Add YARN binaries to root's .bashrc
USER root
RUN echo "" >> ~/.bashrc \
    && echo 'export YARN_DIR="/home/${HOST_USERNAME}/.yarn"' >> ~/.bashrc \
    && echo 'export PATH="$YARN_DIR/bin:$PATH"' >> ~/.bashrc

# Source the bash - illuminator
USER ${HOST_USERNAME}
RUN . ~/.bashrc \
    && yarn global add gulp bower vue-cli

# Source the bash - root
USER root
RUN . ~/.bashrc \
    && yarn global add gulp bower vue-cli



#####################################
# Docker Entrypoint:
#####################################
USER root
COPY ./scripts/entrypoint/ /usr/local/bin/

ENTRYPOINT ["/sbin/tini", "--", "docker-entrypoint"]

WORKDIR /var/www/html

CMD ["/bin/bash", "--login"]
