Nginx [engine x] is an HTTP and reverse proxy server, as well as a mail proxy server, written by Igor Sysoev. Due to its high performance out-of-the-box, it's become one of the world's most popular web servers.

## Add Nginx to Sources

As per the official Nginx docs (http://nginx.org/en/download.html).

    sudo vi /etc/apt/sources.list

Add the following lines to the end of file:

    deb http://nginx.org/packages/ubuntu/ lucid nginx
    deb-src http://nginx.org/packages/ubuntu/ lucid nginx

## Add the Key

    sudo wget http://nginx.org/keys/nginx_signing.key
    sudo apt-key add nginx_signing.key
    sudo aptitude update

## Install Nginx

The -v switch shows the Nginx version number.

    sudo aptitude install nginx
    sudo nginx -v

## Configure Nginx

Check how many cores our server is running and configure Nginx.

    grep processor /proc/cpuinfo
    sudo vi /etc/nginx/nginx.conf

This configuration is based on 4 cores, which we'll map to worker_processes. The default setting for gzip_types has a MIME type error, so replace as per below.

    worker_processes 4;
    worker_connections 768;
    keepalive_timeout 3;
    gzip on;
    gzip_disable "msie6";
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/xml text/css text/javascript application/json application/javascript application/x-javascript application/xml application/xhtml+xml application/rss+xml;

## Configure Virtual Hosts

Create hosting folders and a config file for our virtual host.

    sudo mkdir -p /srv/www/example.com/web
    sudo mkdir -p /srv/www/example.com/logs
    sudo vi /etc/nginx/sites-available/example.com

Listen on port 80, redirect www, and set paths.

    index index.htm index.html index.php;
    server {
        listen 80;
        server_name www.example.com;
        rewrite ^ $scheme://example.com$request_uri? permanent;
    }
    server {
        listen 80;
        server_name example.com;
        access_log  /srv/www/example.com/logs/access.log;
        error_log   /srv/www/example.com/logs/error.log;
        root /srv/www/example.com/web;
    }

## Enable the Website

Create a symbolic link between our site and enabled sites. Test the config file using the -t switch and restart Nginx. Once live, use reload instead of restart to reduce downtime.

    sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled
    sudo nginx -t
    sudo service nginx restart
