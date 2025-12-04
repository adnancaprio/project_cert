
FROM php:8.2-apache


RUN a2enmod rewrite


RUN docker-php-ext-install pdo pdo_mysql


WORKDIR /var/www/html


COPY . /var/www/html/

RUN sed -i 's/AllowOverride None/AllowOverride All/g' /etc/apache2/apache2.conf

EXPOSE 80

CMD ["apache2-foreground"]
