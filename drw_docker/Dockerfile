FROM ubuntu:16.04
MAINTAINER Jan Noreikat <jan.noreikat@cocomore.com>
ENV DEBIAN_FRONTEND noninteractive
RUN rm /bin/sh && ln -s /bin/bash /bin/sh
RUN export TERM=xterm

# Install packages.
RUN apt-get update
RUN apt-get install -y \
	vim \
	git \
	apache2 \
	php-cli \
	php-mysql \
	php-gd \
	php-curl \
	php-mbstring \
	php-xml \
	php-xdebug \
	libapache2-mod-php \
	curl \
	mysql-client \
	openssh-server \
	unzip \
	lftp \
	wget \
    bzip2 \
	supervisor
RUN apt-get clean

# Install Composer.
RUN curl -sS https://getcomposer.org/installer | php
RUN mv composer.phar /usr/local/bin/composer

# Install Drush 8.
RUN composer global require drush/drush:8.*
RUN composer global update
# Unfortunately, adding the composer vendor dir to the PATH doesn't seem to work. So:
RUN ln -s /root/.composer/vendor/bin/drush /usr/local/bin/drush

# Install Drupal Console.
RUN curl http://drupalconsole.com/installer -L -o drupal.phar
RUN mv drupal.phar /usr/local/bin/drupal && chmod +x /usr/local/bin/drupal

# Setup PHP.
RUN sed -i 's/display_errors = Off/display_errors = On/' /etc/php/7.0/apache2/php.ini
RUN sed -i 's/display_errors = Off/display_errors = On/' /etc/php/7.0/cli/php.ini

# Setup Apache.
# In order to run our Simpletest tests, we need to make Apache
# listen on the same port as the one we forwarded. Because we use
# 8080 by default, we set it up for that port.
RUN sed -i 's/AllowOverride None/AllowOverride All/' /etc/apache2/apache2.conf
RUN sed -i 's/DocumentRoot \/var\/www\/html/DocumentRoot \/var\/www/' /etc/apache2/sites-available/000-default.conf
#RUN sed -i 's/VirtualHost \*:80/VirtualHost \*:\*/' /etc/apache2/sites-available/000-default.conf
RUN a2enmod rewrite

COPY assets/apache/drw.conf /etc/apache2/sites-enabled/drw.conf

# Setup xDebug
RUN echo "# xDebug configurations: " >> /etc/php/7.0/apache2/php.ini
RUN echo "zend_extension=$(find /usr/lib/php/7.0/ -name xdebug.so)" >> /etc/php/7.0/apache2/php.ini
RUN echo "xdebug.remote_enable=on" >> /etc/php/7.0/apache2/php.ini
RUN echo "xdebug.remote_autostart=off" >> /etc/php/7.0/apache2/php.ini
# xdebug.max_nesting_level=256 needs to be at cli and apache2
RUN echo "xdebug.max_nesting_level=256" >> /etc/php/7.0/mods-available/xdebug.ini
RUN echo "xdebug.remote_connect_back=1" >> /etc/php/7.0/mods-available/xdebug.ini
RUN echo "xdebug.show_error_trace=1" >> /etc/php/7.0/mods-available/xdebug.ini

# Enable SSL
RUN a2enmod ssl

# Setup SSH.
RUN echo 'root:root' | chpasswd
RUN sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN mkdir /var/run/sshd && chmod 0755 /var/run/sshd
RUN mkdir -p /root/.ssh/ && touch /root/.ssh/authorized_keys
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd

# Install Phantomjs
RUN mkdir /tmp/phantomjs \
    && curl -L https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-2.1.1-linux-x86_64.tar.bz2 \
        | tar -xj --strip-components=1 -C /tmp/phantomjs \
    && mv /tmp/phantomjs/bin/phantomjs /usr/local/bin

# Setup Supervisor.
RUN echo -e '[program:apache2]\ncommand=/bin/bash -c "source /etc/apache2/envvars && exec /usr/sbin/apache2 -DFOREGROUND"\nautorestart=true\n\n' >> /etc/supervisor/supervisord.conf
RUN echo -e '[program:sshd]\ncommand=/usr/sbin/sshd -D\n\n' >> /etc/supervisor/supervisord.conf

# MAC OS X changes
RUN usermod -u 1000 www-data
RUN usermod -a -G users www-data
RUN chown -R www-data:www-data /var/www

# Install nodejs and NPM like frontend does.
RUN groupadd --gid 1001 node \
  && useradd --uid 1001 --gid node --shell /bin/bash --create-home node

# gpg keys listed at https://github.com/nodejs/node#release-team
RUN set -ex \
  && for key in \
    9554F04D7259F04124DE6B476D5A82AC7E37093B \
    94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
    FD3A5288F042B6850C66B31F09FE44734EB7990E \
    71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
    DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
    B9AE9905FFD7803F25714661B63B535A4C206CA9 \
    C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
    56730D5401028683275BD23C23EFEFE93C4CFFFE \
  ; do \
    gpg --keyserver pgp.mit.edu --recv-keys "$key" || \
    gpg --keyserver keyserver.pgp.com --recv-keys "$key" || \
    gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key" ; \
  done

ENV NPM_CONFIG_LOGLEVEL info
ENV NODE_VERSION 7.4.0

RUN buildDeps='xz-utils' \
    && set -x \
    && apt-get update && apt-get install -y $buildDeps --no-install-recommends \
    && rm -rf /var/lib/apt/lists/* \
    && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
    && curl -SLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
    && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
    && grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
    && tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components=1 \
    && rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
    && apt-get purge -y --auto-remove $buildDeps \
    && ln -s /usr/local/bin/node /usr/local/bin/nodejs

RUN npm install -g bower

EXPOSE 80 22 8910 3000
CMD exec supervisord -n

