#######################
# Build backend source
#######################
FROM composer:2.5.1 as backend
WORKDIR /app

# Copy composer.lock and composer.json
COPY composer.json composer.lock /app/

# Composer install
RUN composer install  \
    --ignore-platform-reqs \
    --no-ansi \
    --no-autoloader \
    --no-dev \
    --no-interaction \
    --no-scripts

COPY . /app/
RUN composer dump-autoload --optimize --classmap-authoritative

#####################
# Build the frontend
#####################
FROM node as frontend

RUN mkdir -p /app/public

COPY package.json package-lock.json webpack.mix.js /app/
WORKDIR /app

RUN npm install
# RUN npm run production

####################
# Build the image
####################
FROM php:8-fpm-buster

# Set working directory
WORKDIR /var/www

# Install dependencies
RUN apt-get update && apt-get install -y \
    openssl \
    zip \
    unzip \
    git \
    nodejs \
    libmagickwand-dev  \
    default-mysql-client \
    iputils-ping \
    libonig-dev \
    libzip-dev \
    libpng-dev \
    libjpeg-dev

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*


# Install / configure GD
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install gd

# Install & enable Imagick:
RUN pecl install imagick && \
    docker-php-ext-enable imagick

# Install extensions
RUN docker-php-ext-install mbstring pdo pdo_mysql pcntl zip bcmath

COPY --from=backend /app /var/www
COPY --from=frontend /app/node_modules /var/www/node_modules

# Copy existing application directory permissions
COPY --chown=${UID}:${GID} . /var/www

# Change current user
USER ${UID}

# Expose port 9000 and start php-fpm server
EXPOSE 9000
CMD ["php-fpm"]