FROM php:8.1.28-fpm

# Installation des dépendances
RUN apt-get update && apt-get install -y libpq-dev curl
RUN docker-php-ext-install pdo pdo_mysql

# Copier les fichiers de l'application
COPY k8s-base-app /var/www/html

# Définir le répertoire de travail
WORKDIR /var/www/html

# Installer Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
RUN php -r "if (hash_file('sha384', 'composer-setup.php') === 'dac665fdc30fdd8ec78b38b9800061b4150413ff2e3b6f88543c636f7cd84f6db9189d43a81e5503cda447da73c7e5b6') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
RUN php composer-setup.php
RUN php -r "unlink('composer-setup.php');"
RUN curl -1sLf 'https://dl.cloudsmith.io/public/symfony/stable/setup.deb.sh' | bash
RUN apt install -y symfony-cli

# Exposer le port
EXPOSE 8000

CMD ["symfony", "server:start", "--no-tls"]
