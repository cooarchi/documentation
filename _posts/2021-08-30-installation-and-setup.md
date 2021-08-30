# cooArchi Installation/Setup

This document tries to help you to get cooArchi up and running on your webserver.

## Requirements

- A webserver, virtual or bare metal - Hetzner Cloud or DigitalOcean Droplet works fine. OS is your choice. Ubuntu is not a bad decision in sense of documentation available in the internet. But every other OS will work, too. 
- Web Server of your choice: Nginx, Apache, Caddy
- PHP 7.4
- MySQL Database (with own DB and user)
- A domain you want to run cooArchi on with DNS settings already pointing to your server

As these are base requirements, we decided to point you to another tutorial to get this stuff prepared. Follow [Erika Heidi's superb tutorial about the LEMP stack](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-20-04) and come back to this tutorial, if you're finished and the requirements are ready.


## Install cooArchi

There are two possible ways of installing cooArchi. 

First one: Install the release tar ball from GitHub (`https://github.com/cooarchi/cooarchi/releases`) on your server. This package already contains vendor libraries and the visualisation frontend code. Just upload the tar file to your webserver (`/var/www/cooarchi` if you followed Erika Heidi's LEMP tutorial), unpack it and go on with step `Configure cooArchi`

Second one: Create your personal fork of [cooArchi](https://github.com/cooarchi/cooarchi). Add a deploy token for your fork and clone the repository to your webserver.

```bash
$ git clone <url> /var/www/cooarchi
```

Fork `https://github.com/cooarchi/cooarchi-ui`, too and clone your fork into `/var/www/cooarchi/public/ui` folder after that.

Install [composer](https://getcomposer.org) global or download phar inside project folder.

Run following commands then:

```bash
$ php composer.phar install
```

This will install all needed vendor libraries through packagist.


## Configure cooArchi

To get your cooArchi installation up and running, following steps are needed:

```bash
$ vendor/bin/laminas cooArchi:setup
```

Use the DB credentials you created inside the tutorial. It is not a bad idea to have a separate DB user for cooarchi with permission only for your cooArchi database.


```bash
$ vendor/bin/laminas cooArchi:create-administrata
```

Write down the credentials - you need them later.


```bash
$ vendor/bin/doctrine orm:generate-proxies
```

```bash
$ chmod 777 public/files
```

```bash
$ vendor/bin/doctrine orm:schema-tool:update --dump-sql
```

Copy SQL queries and execute them inside your MySQL database.


## Configure Webserver

We provide a Nginx setup example here. You can use any other Webserver. We do not use any special configuration settings. Only `client_max_body_size 100m` is used to allow large file uploads inside cooArchi.

```
server {
    listen 80;
    listen [::]:80;
    
    server_name foo.cooarchi.net;

    location / {
        root /var/www/foo.cooarchi.net/public;
        try_files $uri $uri/ /index.php?$args;
        index index.html index.php;
        client_max_body_size 100m;

        location ~ \.(jp(e)?g|gif|css|png|js)$ {
            access_log off;
            expires 30d;
        }

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
            client_max_body_size 100m;
        }
    }

    location ~ /\.ht {
        deny all;
    }
}

```

## SSL

Use `certbot` (https://letsencrypt.org/de/getting-started/) to get SSL up and running for your installation. `certbot` will do redirect magic part by itself, so you don't need to care about that part.

`certbot --nginx -d foo.cooarchi.net`

## PHP Settings

To enable large file uploads, you need to edit you `php.ini`. You will find it inside `/etc/php/7.4/fpm/php.ini`.

Change following values:

`post_max_size = 100M` and 
`upload_max_filesize = 100M`

## Restart Nginx and PHP

- `systemctl restart nginx.service`
- `systemctl restart php7.4-fpm.service`

Verify running status by:

- `systemctl status nginx.service`
- `systemctl status php7.4-fpm.service`

## Test your installation

With your browser: visit `foo.cooarchi.net`. You should see the cooArchi startpage. 

Login with your administratA credentials. Create your first invitation: `https://foo.cooarchi.net/invitations` and try to create your first element/relation combination via the visualisation. Testing file upload is not a bad idea, too. 

If everything works fine - great! Enjoy your own cooArchi installation and spread the archiving love further.

Send an email to `info [ at ] cooarchi [ dot ] net` about your installation, so we can list it on our own website, if you want to.
