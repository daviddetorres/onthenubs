FROM php:7-apache

COPY 000-default.conf /etc/apache2/sites-available/000-default.conf
COPY start-apache.sh /usr/local/bin
RUN a2enmod rewrite

COPY index.php /var/www/
RUN chown -R www-data:www-data /var/www

EXPOSE 80

CMD [ "bash", "start-apache.sh" ]
