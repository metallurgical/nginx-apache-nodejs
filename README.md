## Introduction

Setting up nginx as reverse proxy that forward traffic to apache web server to serve PHP and nodejs application using different port. Self reference only easy for me to refer next time when needed.

## Requirements
- nginx
- php (Using php 7.2)
- apache
- nodejs simple application run on `port: 3000`
- PHP application run on `port :8080`

## Installations

### Install Apache

Update local package index
```
sudo apt update
```

Install apache2 package
```
sudo apt install apache2
```

Change default port from `80` to `8080`
```
sudo vim /etc/apache2/ports.conf

# change Listen 80 into Listen 8080. Save and close vim editor
```

Restart apache2
```
sudo systemctl restart apache2
```

stop apache2
```
sudo systemctl stop apache2
```

Check status
```
sudo systemctl status apache2
```

Reload apache2
```
sudo systemctl reload apache2
```

Disable apache2 from startup boot
```
sudo systemctl disable apache2
```

Enable apache2 to start on startup boot
```
sudo systemctl enable apache2
```

### Install PHP (7.2)

```
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
sudo apt-get install -y php7.2
```

Check version
```
php -v
```

Install common modules
```
sudo apt-get install php7.2-mysql php7.2-curl php7.2-zip php7.2-json php7.2-cgi php7.2-xsl
```

Change common config
```
sudo vim /etc/php/7.2/apache2/php.ini

## Find this setting and change to
file_uploads = On
allow_url_fopen = On
memory_limit = 256M
upload_max_filesize = 100M
max_execution_time = 360
```

### Install Nginx

Update local package index
```
sudo apt update
```

Install nginx package
```
sudo apt install nginx
```

Restart nginx
```
sudo systemctl restart nginx
```

Stop nginx
```
sudo systemctl stop nginx
```

Check status
```
sudo systemctl status nginx
```

Reload nginx
```
sudo systemctl reload nginx
```

Disable nginx from startup boot
```
sudo systemctl disable nginx
```

Enable nginx to start on startup boot
```
sudo systemctl enable nginx
```

### Install Nodejs/Npm

Update local package index
```
sudo apt update
```

Install nodejs package
```
sudo apt install nodejs
```

Install npm
```
sudo apt install npm
```

### Create simple nodejs application
Create simple nodejs application which run on `port :3000`
```
mkdir /var/www/node-simple-app
cd /var/www/node-simple-app
vim app.js
```

Put following code
```javascript
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => res.send('Hello World!'))

app.listen(port, () => console.log(`Example app listening on port ${port}!`))
```

Save and closed. Run
```
npm init
npm install --save express
```

Run application. Make sure to always run this application to make `port :3000` always open.
```
node app.js
```

### Create simple PHP application
Create simple nodejs application which run on `port :8080`
```
mkdir /var/www/php-simple-app
cd /var/www/php-simple-app
vim index.php
```

Put following code
```php
<?php
phpinfo();
?>
```

Save and closed. Disable default vhost
```
cd /etc/apache2/sites-available
sudo a2dissite 000-default.conf
```

Create new vhost
```
sudo cp 000-default.conf php-simple-app.conf
vim php-simple-app.conf
```

Put following code 
```
<VirtualHost *:8080>
        ServerName 167.71.211.151 # Remote ip address

        ServerAdmin norlihazmey.ghazali@gmail.com
        DocumentRoot /var/www/php-simple-app

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Enable new vhost
```
sudo a2ensite php-simple-app.conf
```

Reload apache2
```
sudo systemctl reload apache2
```

### Reverse Proxy from nginx to PHP site
Pass traffic from `port :80` nginx into `port :8080` of PHP app. Create nginx server block
```
vim /etc/nginx/sites-available/php-simple-app.conf
```

Put following basic content
```
server {
        listen 80;
        listen [::]:80;

        server_name php-test.example.com;

        location / {

                proxy_pass_header Authorization;
                proxy_pass http://167.71.211.151:8080; # pass traffic to apache with listen to port 8080
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_http_version 1.1;
                proxy_set_header Connection "";
                proxy_buffering off;
                client_max_body_size 0;
                proxy_read_timeout 36000s;
                proxy_redirect off;
        }
}
```

### Reverse Proxy from nginx to Node site
Pass traffic from `port :80` nginx into `port :3000` of nodejs. Create nginx server block
```
vim /etc/nginx/sites-available/node-simple-app.conf
```

Put following basic content
```
server {
        listen 80;
        listen [::]:80;

        server_name node-test.example.com;

        location / {

                proxy_pass_header Authorization;
                proxy_pass http://167.71.211.151:3000; # pass traffic to apache with listen to port 8080
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_http_version 1.1;
                proxy_set_header Connection "";
                proxy_buffering off;
                client_max_body_size 0;
                proxy_read_timeout 36000s;
                proxy_redirect off;
        }
}
```

Disable default server block
```
sudo mv /etc/nginx/sites-available/default /etc/nginx/sites-available/default.disabled 
```

Delete default server block
```
sudo rm /etc/nginx/sites-enabled/default
```

Enable PHP apps nginx server block
```
sudo ln -s /etc/nginx/sites-available/php-simple-app.conf /etc/nginx/sites-enabled/php-simple-app.conf 
```

Enable Node apps nginx server block
```
sudo ln -s /etc/nginx/sites-available/node-simple-app.conf /etc/nginx/sites-enabled/node-simple-app.conf 
```

Test nginx config
```
nginx -t
```

Restart nginx
```
sudo systemctl restart nginx
```

## Test
When navigate to `http://php-test.example.com`, nginx will redirect traffic to `http://<localhost|server-ip>:8080` which serve the PHP site.

When navigate to `http://node-test.example.com`, nginx will redirect traffic to `http://<localhost|server-ip>:3000` which serve the Node JS site.



