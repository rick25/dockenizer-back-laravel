mkdir -p mysql
mkdir -p nginx/conf
touch nginx/conf/app.conf
server {
listen 80;
index index.php index.html;
error_log /var/log/nginx/error.log;
access_log /var/log/nginx/access.log;
root /var/www/public;
location ~ \.php$ {
try_files $uri =404;
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
fastcgi_pass app:9000;
fastcgi_index index.php;
include fastcgi_params;
fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
fastcgi_param PATH_INFO $fastcgi_path_info;
        }
        location / {
            try_files $uri $uri/ /index.php?$query_string;
gzip_static on;
}
}
git clone https://github.com/laravel/laravel.git src

touch docker-compose.yml
version: "3.8"

    services:
    app:
        build:
        context: ./
        dockerfile: Dockerfile
        image: laravel8-php-fpm-80
        container_name: app
        restart: unless-stopped
        tty: true
        working_dir: /var/www
        volumes:
        - ./src:/var/www
        networks:
        - app-network

    mysql:
        image: mysql:8.0.27
        container_name: mysql
        restart: unless-stopped
        tty: true
        environment:
        MYSQL_DATABASE: impresiones
        MYSQL_ROOT_PASSWORD: 123456
        MYSQL_PASSWORD: 123456
        MYSQL_USER: ricardo
        SERVICE_TAGS: dev
        SERVICE_NAME: mysql
        volumes:
        - ./mysql/data:/var/lib/mysql
        networks:
        - app-network

    nginx:
        image: nginx:1.20.2-alpine
        container_name: nginx
        restart: unless-stopped
        tty: true
        ports:
        - 8100:80
        volumes:
        - ./src:/var/www
        - ./nginx/conf:/etc/nginx/conf.d
        networks:
        - app-network

    networks:
    app-network:
        driver: bridge

touch Dockerfile
FROM php:8.0.3-fpm-buster

    RUN docker-php-ext-install bcmath pdo_mysql
    RUN apt-get update
    RUN apt-get install -y git zip unzip

    COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

    EXPOSE 9000

touch .gitignore
mysql
src
docker-compose exec -T app composer install
docker-compose exec -T app cp .env.example .env
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=laravel8
DB_USERNAME=ricardo
DB_PASSWORD=123456
docker-compose exec app chown -R www-data:www-data /var/www/storage
docker-compose exec app chown -R www-data:www-data /var/www/bootstrap/cache
docker-compose exec -T app php artisan key:generate
