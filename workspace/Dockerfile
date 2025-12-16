FROM phusion/baseimage:noble-1.0.2

ARG PHP_VERSION=8.4

# Set environment
ENV DEBIAN_FRONTEND=noninteractive \
    TERM=xterm

USER root

ARG PUID=1000
ENV PUID=${PUID}
ARG PGID=1000
ENV PGID=${PGID}

RUN set -xe; \
    groupmod --new-name localdev ubuntu; \
    usermod --login localdev ubuntu --move-home --home /home/localdev; \
    usermod -p "*" localdev -s /bin/bash

# Install base packages
RUN apt-get update && apt-get install -y \
    apt-utils \
    curl \
    wget \
    git \
    nano \
    vim \
    htop \
    sudo \
    gnupg \
    lsb-release \
    ca-certificates \
    apt-transport-https \
    software-properties-common \
    zip \
    && rm -rf /var/lib/apt/lists/*

# Add PHP repository and install PHP
RUN add-apt-repository ppa:ondrej/php -y && \
    apt-get update && \
    apt-get install -y --allow-downgrades --allow-remove-essential \
    --allow-change-held-packages \
    php${PHP_VERSION}-dev \
    php${PHP_VERSION}-cli \
    php${PHP_VERSION}-common \
    php${PHP_VERSION}-bcmath \
    php${PHP_VERSION}-bz2 \
    php${PHP_VERSION}-calendar \
    php${PHP_VERSION}-curl \
    php${PHP_VERSION}-dba \
    php${PHP_VERSION}-exif \
    php${PHP_VERSION}-ffi \
    php${PHP_VERSION}-ftp \
    php${PHP_VERSION}-gd \
    php${PHP_VERSION}-gmp \
    php${PHP_VERSION}-igbinary \
    php${PHP_VERSION}-imagick \
    php${PHP_VERSION}-imap \
    php${PHP_VERSION}-intl \
    php${PHP_VERSION}-ldap \
    php${PHP_VERSION}-lz4 \
    php${PHP_VERSION}-mbstring \
    php${PHP_VERSION}-mongodb \
    php${PHP_VERSION}-mysqli \
    php${PHP_VERSION}-pgsql \
    php${PHP_VERSION}-redis \
    php${PHP_VERSION}-shmop \
    php${PHP_VERSION}-soap \
    php${PHP_VERSION}-sockets \
    php${PHP_VERSION}-sqlite3 \
    php${PHP_VERSION}-sysvmsg \
    php${PHP_VERSION}-sysvsem \
    php${PHP_VERSION}-sysvshm \
    php${PHP_VERSION}-xsl \
    php${PHP_VERSION}-zip \
    && rm -rf /var/lib/apt/lists/*


# Install additional development tools
RUN apt-get update && apt-get install -y \
    mysql-client \
    postgresql-client \
    redis-tools \
    && rm -rf /var/lib/apt/lists/*

############################################
# Composer
############################################

USER root

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Add the composer.json
COPY ./composer.json /home/localdev/.composer/composer.json

# Make sure that ~/.composer belongs to localdev
RUN chown -R localdev:localdev /home/localdev/.composer

USER localdev

# Export composer vendor path
RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="$HOME/.composer/vendor/bin:$PATH"' >> ~/.bashrc

############################################
# NVM
############################################

USER localdev

ENV NVM_DIR=/home/localdev/.nvm
ENV NPM_FETCH_RETRIES=2
ENV NPM_FETCH_RETRY_FACTOR=10
ENV NPM_FETCH_RETRY_MINTIMEOUT=10000
ENV NPM_FETCH_RETRY_MAXTIMEOUT=60000

# Install nvm (A Node Version Manager)
RUN mkdir -p $NVM_DIR && \
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash \
    && . $NVM_DIR/nvm.sh \
    && nvm install node \
    && nvm use node \
    && nvm alias node \
    && npm cache clear --force \
    && npm config set fetch-retries ${NPM_FETCH_RETRIES} \
    && npm config set fetch-retry-factor ${NPM_FETCH_RETRY_FACTOR} \
    && npm config set fetch-retry-mintimeout ${NPM_FETCH_RETRY_MINTIMEOUT} \
    && npm config set fetch-retry-maxtimeout ${NPM_FETCH_RETRY_MAXTIMEOUT}

RUN echo "" >> ~/.bashrc && \
    echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc && \
    echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc

# Add NVM binaries to root's .bashrc
USER root

RUN echo "" >> ~/.bashrc && \
    echo 'export NVM_DIR="/home/localdev/.nvm"' >> ~/.bashrc && \
    echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc

RUN . "$NVM_DIR/nvm.sh"

# Make it so the node modules can be executed with 'docker-compose exec'
# We'll create symbolic links into '/usr/local/bin'.
RUN find $NVM_DIR -type f -name node -exec ln -s {} /usr/local/bin/node \; && \
    NODE_MODS_DIR="$NVM_DIR/versions/node/$(node -v)/lib/node_modules" && \
    ln -s $NODE_MODS_DIR/npm/bin/npm-cli.js /usr/local/bin/npm && \
    ln -s $NODE_MODS_DIR/npm/bin/npx-cli.js /usr/local/bin/npx

############################################
# Non-root user : PHPUnit path
############################################

# add ./vendor/bin to non-root user's bashrc (needed for phpunit)
USER localdev

RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="/var/www/vendor/bin:$PATH"' >> ~/.bashrc

############################################
# Crontab
############################################

USER root

COPY ./crontab /etc/cron.d

RUN chmod -R 644 /etc/cron.d

############################################
# Set Aliases
############################################

USER root

COPY ./aliases.sh /root/aliases.sh
COPY ./aliases.sh /home/localdev/aliases.sh

RUN sed -i 's/\r//' /root/aliases.sh && \
    sed -i 's/\r//' /home/localdev/aliases.sh && \
    chown localdev:localdev /home/localdev/aliases.sh && \
    echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
    echo "" >> ~/.bashrc

USER localdev

RUN echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
    echo "" >> ~/.bashrc

############################################
# Final Touch
############################################

USER root

# Create project directory and set permissions
# RUN mkdir -p /var/www && \
#     chown localdev:localdev /var/www

# Clean up APT
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Set working directory
WORKDIR /var/www
