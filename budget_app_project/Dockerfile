# Dockerfile
FROM node:16.20.0 AS node
FROM php:8.0.27-fpm
COPY --from=node /usr/local/lib/node_modules /usr/local/lib/node_modules
COPY --from=node /usr/local/bin/node /usr/local/bin/node
RUN ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm
#ensure we are ROOT for the install
USER root
# Copy composer.lock and composer.json
COPY composer.lock composer.json /app/
#copy the ENV file
COPY .env /app/.env
# Set working directory
WORKDIR /app/
# Install dependencies
RUN apt-get update && apt-get install -y \
    libmemcached-dev \
    build-essential \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    locales \
    zip \
    libzip-dev \
    jpegoptim optipng pngquant gifsicle \
    vim \
    unzip \
    git \
    curl \
    cifs-utils \
    keyutils
# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*
# Install extensions
RUN docker-php-ext-install pdo_mysql zip exif pcntl
RUN docker-php-ext-configure gd --with-freetype --with-jpeg
RUN docker-php-ext-install gd
RUN docker-php-ext-install opcache
RUN docker-php-ext-install bcmath
RUN docker-php-ext-enable bcmath
RUN docker-php-ext-install mysqli
RUN docker-php-ext-enable mysqli
# Add user for laravel application
RUN groupadd -g 1000 www
RUN useradd -u 1000 -ms /bin/bash -g www www
# Copy existing application directory contents
COPY . /app/
# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
#List Config
RUN npm config ls -l
# Run NPM
RUN npm install
RUN npm run dev
RUN composer install --optimize-autoloader
RUN php artisan route:clear
RUN php artisan route:cache
RUN php artisan config:clear
RUN php artisan config:cache
RUN php artisan storage:link
RUN composer dumpautoload -o
# Copy existing application directory permissions
COPY --chown=www:www . /app/
# mkdir for data
RUN mkdir -p /data/
RUN chmod -R 777 /data/
# Change current user to www
USER www
# Expose port 9000 and start php-fpm server
EXPOSE 9000
CMD ["php-fpm"]
#Copy
RUN cp -r /app/public /var/www/html/