# Debian10-Lemp-Laravel
Setup Nginx, PHP 8, MariaDB, Laravel 8 and Let's Encrypt on Debian 10

**Tested on Hetzner VPS**

* Create new Debian 10 instance.

* Connect to your  VPS via ssh using the credential you received by email

    ```
    ssh root@IP_OF_YOUR_VPS
    ```

* Upgrade the system

    ```
    apt update
    apt upgrade
    ```

* Enter `Y` if/when asked.

* Install [Nginx](https://www.nginx.com/)

    ```
    apt install nginx
    ```

* Install [MariaDB](https://mariadb.org/)

    ```
    apt install mariadb-server
    ```

* Connect to MariaDB

    ```
    mariadb
    ```

* Create database

    ```
    create database DATABASE_NAME;
    ```

* Create a new database user

    ```
    CREATE USER YOUR_DATABASE_USER IDENTIFIED BY 'YOUR_DATABASE_USER_PASSWORD';
    ```
* Grant privileges to this user

    ```
    GRANT ALL ON DATABASE_NAME.* TO YOUR_DATABASE_USER;
    ```

* Exit from MariaDB

    ```
    exit
    ```
    
* Secure your MariaDB installation

    ```
    mysql_secure_installation
    ```

* Add [Sury](https://deb.sury.org/) repository to install [PHP 8](https://www.php.net/).

    ```
    apt -y install lsb-release apt-transport-https ca-certificates 
    ```

    ```
    sudo wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
    ```

    ```
    echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/php.list
    ```

* Upgrade the system

    ```
    apt update
    apt upgrade
    ```

* Install PHP and extensions

    ```
    apt install php8.0-fpm php8.0-mysql php8.0-mbstring php8.0-xml php8.0-bcmath zip unzip curl git
    ```
    
* Install [Composer](https://getcomposer.org/) 

    ```
    curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
    ```
    
* Go to www directory

    ```
    cd /var/www/
    ```
    
* Install [Laravel](https://laravel.com/)

    ```
    composer create-project --prefer-dist laravel/laravel laravel
    ```

* Set right permissions

    ```
    chown -R www-data:www-data /var/www/laravel/
    chmod -R 755 /var/www/laravel/
    ```

* Edit Laravel configuration settings and update DB connection variables

    ```
    nano .env
    ```

    ```
    DB_DATABASE=YOUR_DATABASE_NAME
    DB_USERNAME=YOUR_DATABASE_USER
    DB_PASSWORD=YOUR_DATABASE_USER_PASSWORD
    ```

* Edit Nginx configuration to use Laravel

    ```
    nano /etc/nginx/sites-available/default.conf
    ```
    
    ```
    server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/laravel/public;
        
        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Content-Type-Options "nosniff";

        index index.html index.htm index.php;

        charset utf-8;

        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        location = /favicon.ico { access_log off; log_not_found off; }
        location = /robots.txt  { access_log off; log_not_found off; }

        error_page 404 /index.php;

        location ~ \.php$ {
            fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
            include fastcgi_params;
        }

        location ~ /\.(?!well-known).* {
            deny all;
        }
    }
    ```

* Test Nginx configuration

    ```
    nginx -t
    ```

* Reload Nginx

    ```
    service nginx reload
    ```

* Install [Node.js](https://nodejs.org/en/) and [npm](https://www.npmjs.com/)

    ```
    ccurl -fsSL https://deb.nodesource.com/setup_17.x | bash -
    apt-get install -y nodejs
    ```


* Add Cron task

    ```
    crontab -l | { cat; echo "* * * * * php /var/www/laravel/artisan schedule:run >> /dev/null 2>&1"; } | crontab -
    ```
    
* Check Cron file

     ```
    more /var/spool/cron/crontabs/root
    ```

* Install [Cerbot](https://certbot.eff.org/) to obtain your SSL certificate (you need a domain)

    ```
    apt install snapd
    snap install core; sudo snap refresh core
    snap install --classic certbot
    ln -s /snap/bin/certbot /usr/bin/certbot
    certbot --nginx
    ```

* Your are up&running, Laravel is installed with LEMP and PHP8




## Reference
- mezhevikin on [GitHub](https://github.com/mezhevikin/lemp-laravel-debian)
- Rahul Bagul on [Medium](https://medium.com/@rahulbagul330/how-to-install-laravel-php-framework-with-nginx-on-debian-10-linux4one-81843c827b29)
- Shahriar Shovon on [Linux Hint](https://linuxhint.com/laravel_dev_debian_10/)







