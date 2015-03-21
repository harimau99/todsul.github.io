PHP-FPM (FastCGI Process Manager) is an FastCGI implementation with additional features especially useful for high-load websites. It makes running PHP particularly easy on Nginx.

## Add Repository

Use the latest package with FPM compiled into PHP5.

```
sudo add-apt-repository ppa:ondrej/php5
```

## Install the Package

The -v switch confirms PHP version.

```
sudo aptitude update
sudo aptitude install php5-fpm
php5-fpm -v
```

Install other required PHP packages.

```
sudo aptitude install php5-common
sudo aptitude install php5-gd
sudo aptitude install php5-cli
sudo aptitude install php5-curl
```

## Configure FPM

The Nginx PHP5 package puts the FPM config in a custom location.

```
sudo vi /etc/php5/fpm/pool.d/www.conf
```

Each process takes about 35Mb. Keep this in mind when configuring FPM. These settings are for a 512MB Linode. Adjust as necessary.

```
pm = dynamic
pm.max_children = 4
pm.start_servers = 1
pm.min_spare_servers = 1
pm.max_spare_servers = 2
pm.max_requests = 500
listen = 127.0.0.1:9000
```

Open the FPM config file.

```
sudo vi /etc/php5/fpm/php.ini
```

Set charset to UTF8.

```
default_charset = "utf-8"
```

## Configure FastCGI

The package provides default FastCGI parameters.

```
sudo vi /etc/nginx/fastcgi_params
```

Add the script filename to this file.

```
fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
```

## Configure Nginx

Tell Nginx to pass PHP scripts to FPM.

```
sudo vi /etc/nginx/sites-available/example.com
```

The first try_files command supports pretty URLs by deferring to a front controller. The second try_files command helps thwart arbitrary code execution.

```
index index.php;
server {
    server_name www.example.com;
    rewrite ^ $scheme://example.com$request_uri? permanent;
}
server {
    server_name example.com;
    access_log /srv/www/example.com/logs/access.log;
    error_log /srv/www/example.com/logs/error.log;
    root /srv/www/example.com/web;
    location / {
        try_files $uri $uri/ /index.php;
    }
    location ~ \.php$ {
        try_files $uri = 404;
        include fastcgi_params;
        fastcgi_pass 127.0.0.1:9000;
    }
}
```

## TCP vs Unix Sockets

Unix sockets are typically a little faster, so change if desired.

```
sudo vi /etc/php5/fpm/pool.d/www.conf
sudo vi /etc/nginx/sites-available/example.com
```

Use this line in the FPM config:

```
listen = /var/run/php5-fpm.sock
```

Use this line in the Nginx config:

```
fastcgi_pass unix:/var/run/php5-fpm.sock;
```

## Restart Services

```
sudo service nginx restart
sudo service php5-fpm start
```
