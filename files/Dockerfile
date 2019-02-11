FROM ubuntu:16.04
MAINTAINER Mark Day, mark@mark-day.me
ENV REFRESHED_AT 2019_02_11
ARG DEBIAN_FRONTEND=noninteractive

RUN \
    apt-get update -qq -y && \
    apt-get install -y --no-install-recommends apt-utils \ 
    software-properties-common \
    python-software-properties \
    language-pack-en-base && \
    LC_ALL=en_US.UTF-8 add-apt-repository ppa:ondrej/php && \
    apt-get update && \
    apt-get install -y libapache2-mod-php7.0 php7.0-bcmath php7.0-bz2 php7.0-cli php7.0-common php7.0-curl php7.0-dba php7.0-gd php7.0-gmp php7.0-imap php7.0-intl php7.0-ldap php7.0-mbstring php7.0-mcrypt php7.0-mysql php7.0-odbc php7.0-pgsql php7.0-recode php7.0-snmp php7.0-soap php7.0-sqlite php7.0-tidy php7.0-xml php7.0-xmlrpc php7.0-xsl php7.0-zip php7.0-fpm && \
    apt-get install -y zip curl && \
    apt-get install libapache2-mod-fastcgi && \
    apt-get remove -y python-software-properties software-properties-common && \
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/* && \
    chmod 777 -R /var/www && \
    apache2ctl -t && \
    mkdir -p /run /var/lib/apache2 /var/lib/php && \
    chmod -R 777 /run /var/lib/apache2 /var/lib/php /etc/php/7.0/apache2/php.ini

ADD fqdn.conf /etc/apache2/conf-available/fqdn.conf
LABEL role="webserver"

RUN service apache2 start
RUN a2enmod actions fastcgi alias
RUN a2enconf php7.0-fpm
RUN a2enconf fqdn
RUN a2enmod proxy_fcgi setenvif
RUN service php7.0-fpm restart

ENV APACHE_RUN_USER www-data 
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apache2

VOLUME /var/www/html/
VOLUME /etc/apache2/sites-available/

WORKDIR /var/www/html

EXPOSE 80
EXPOSE 443

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
