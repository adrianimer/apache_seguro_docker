FROM httpd:2.4


RUN apt-get -y update && \
    apt-get -y install --no-install-recommends git \
    libtool \
    dh-autoreconf \
    pkgconf \
    libcurl4-gnutls-dev \
    libxml2 \
    libpcre++-dev \
    libxml2-dev \
    libgeoip-dev \
    libyajl-dev \
    liblmdb-dev \
    ssdeep \ 
    ca-certificates \
    build-essential

RUN cd /opt && \
    git clone -b v3/master https://github.com/SpiderLabs/ModSecurity.git ModSecurity && \
    cd /opt/ModSecurity && \
	./build.sh && \
	git submodule init && \
	git submodule update && \
	./configure --prefix=/usr --with-libmodsecurity=/usr/lib && \
	make && \
	make install

RUN cd /opt && \
    git clone https://github.com/SpiderLabs/ModSecurity-apache ModSecurity-apache && \
    cd /opt/ModSecurity-apache && \
	./autogen.sh && \
	./configure --prefix=/usr --with-libmodsecurity=/usr/lib && \
	make && \
	make install

RUN mkdir -p /usr/local/apache2/modsecurity.d/ && \
    echo "LoadModule security3_module \"$(find /opt/ModSecurity-apache/ -name mod_security3.so)\"" > /usr/local/apache2/conf/extra/security.conf && \
    echo "modsecurity_rules 'SecRuleEngine On'" >> /usr/local/apache2/conf/extra/security.conf && \
	echo "modsecurity_rules_file '/usr/local/apache2/modsecurity.d/include.conf'" >> /usr/local/apache2/conf/extra/security.conf


RUN cd /usr/local/apache2/modsecurity.d && \
    git clone https://github.com/SpiderLabs/owasp-modsecurity-crs owasp-crs && \
    cd /usr/local/apache2/modsecurity.d/ && \
	mv /opt/ModSecurity/modsecurity.conf-recommended /usr/local/apache2/modsecurity.d/modsecurity.conf && \
    echo "include modsecurity.conf" >> /usr/local/apache2/modsecurity.d/include.conf && \
    mv /usr/local/apache2/modsecurity.d/owasp-crs/crs-setup.conf.example /usr/local/apache2/modsecurity.d/owasp-crs/crs-setup.conf && \
    echo "include owasp-crs/crs-setup.conf" >> /usr/local/apache2/modsecurity.d/include.conf && \
	echo "include owasp-crs/rules/*.conf" >> /usr/local/apache2/modsecurity.d/include.conf && \
    sed -i "s/DetectionOnly/On/" /usr/local/apache2/modsecurity.d/modsecurity.conf && \
    sed -i "s@/var/log/modsec_audit.log@/usr/local/apache2/logs/modsec_audit.log@" /usr/local/apache2/modsecurity.d/modsecurity.conf && \
    ln -s /usr/local/apache2/bin/httpd /usr/sbin/httpd


RUN mkdir -p /usr/local/apache2/htdocs/adrianimer && \
    mkdir -p /usr/local/apache2/htdocs/tomchi


COPY ./apache.conf /usr/local/apache2/conf/httpd.conf

RUN chgrp -R www-data /usr/local/apache2/conf/httpd.conf && \
    usermod -u 1000 www-data && \
    groupmod -g 1000 www-data && \
    chgrp -R www-data /usr/local/apache2

EXPOSE 80
EXPOSE 443


