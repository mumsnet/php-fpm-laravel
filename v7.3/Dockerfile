FROM mumsnet/php-fpm:7.3

# Install node and npm, needed by Laravel for asset compilation
RUN curl -sL https://deb.nodesource.com/setup_10.x | bash \
    && apt-get update -qq \
    && apt-get install -y nodejs

# Install Laravel globally
RUN composer global require laravel/installer
