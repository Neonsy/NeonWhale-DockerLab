FROM phpmyadmin/phpmyadmin

COPY ./ssl /certs

# Enable Apache SSL module
RUN a2enmod ssl

# Adjust the default Apache configuration to use SSL
RUN sed -ri -e 's,80,443,' /etc/apache2/sites-available/000-default.conf
RUN sed -i -e '/^<\/VirtualHost>/i SSLEngine on' /etc/apache2/sites-available/000-default.conf
RUN sed -i -e '/^<\/VirtualHost>/i SSLCertificateFile /certs/neonspace.com.crt' /etc/apache2/sites-available/000-default.conf
RUN sed -i -e '/^<\/VirtualHost>/i SSLCertificateKeyFile /certs/neonspace.com.key' /etc/apache2/sites-available/000-default.conf

# Expose port 443 for SSL
EXPOSE 443