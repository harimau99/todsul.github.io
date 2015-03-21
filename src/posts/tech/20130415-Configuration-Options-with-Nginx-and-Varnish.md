Nginx's architecture makes it great for handling high load. While Varnish was designed specifically for speed. Let's take a look at the possible configurations of Nginx and Varnish.

## Possible Configurations

<ol>
<li><a href="#config1">[View]</a> Standalone Nginx (port 80)</li>
<li><a href="#config2">[View]</a> Varnish (80) caching for Nginx (8080)</li>
<li><a href="#config3">[View]</a> Nginx (80) for static, Varnish (6081) caching for Nginx (8080)</li>
<li><a href="#config4">[View]</a> Varnish (80) caching static content with Nginx (8080)</li>
<li><a href="#config5">[View]</a> NEW: Standalone Nginx (80) with FastCGI Caching</li>
<li><a href="#config6">[View]</a> NEW: Nginx using FastCGI with TCP/IP vs Unix Sockets</li>
</ol>

## Base Configuration Files

There are four basic configuration files. For each configuration, only the changes to the base configuration are shown. First, use vi to open the Nginx config.

```
sudo vi /etc/nginx/sites-available/example.com
```

Copy and paste the following:

```
index app.php;
server {
    listen 80;
    server_name example.com;
    root /srv/www/example.com/web;
    location / {
        try_files $uri $uri/ /app.php;
    }
    location ~ \.php$ {
        try_files $uri = 404;
        include fastcgi_params;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
    }
}
```

Open the PHP-FPM config.

```
sudo vi /etc/php5/fpm/pool.d/www.conf
```

Copy and paste the following:

```
pm = dynamic
pm.max_children = 4
pm.start_servers = 1
pm.min_spare_servers = 1
pm.max_spare_servers = 2
pm.max_requests = 500
listen = /var/run/php5-fpm.sock
```

Open the Varnish daemon config.

```
sudo vi /etc/default/varnish
```

Copy and paste the following:

```
DAEMON_OPTS="-a :6081
  -T localhost:6082
  -f /etc/varnish/default.vcl
  -S /etc/varnish/secret
  -s malloc,256M"
```

Open the default Varnish VCL file.

```
sudo vi /etc/varnish/default.vcl
```

Copy and paste the following:

```
backend default {
    .host = "127.0.0.1";
    .port = "8080";
}
sub vcl_recv {
    unset req.http.cookie;
}
sub vcl_fetch {
    unset beresp.http.set-cookie;
}
```

<h2 id="config1">Config 1. Standalone Nginx

Use the base configuration of FastCGI with Unix sockets.

<h2 id="config2">Config 2. Varnish in Front

Base config, but with Varnish in front.

```
sudo vi /etc/nginx/sites-available/example.com
```

Change the port to 8080.

```
index app.php;
server {
    listen 8080;
    ...
}
```

Then edit the Varnish config.

```
sudo vi /etc/default/varnish
```

Change the port to 80.

```
DAEMON_OPTS="-a :80
...
```

<h2 id="config3">Config 3. Nginx Sandwich

Base config, but Nginx on 80 and 8080, with Varnish on 6081 in the middle.

```
sudo vi /etc/nginx/sites-available/example.com
```

Add both 80 and 8080 blocks.

```
index app.php;
server {
    listen 80;
    server_name example.com;
    root /srv/www/example.com/web;
    location / {
        try_files $uri $uri/ /app.php;
    }
    location ~ \.php$ {
        try_files $uri = 404;
        proxy_pass http://127.0.0.1:6081;
    }
}
server {
    listen 8080;
    server_name example.com;
    root /srv/www/example.com/web;
    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass 127.0.0.1:9000;
    }
}
```

<h2 id="config4">Config 4. Varnish Pass Static

Base config, but Varnish on 80 and passing the static files to Nginx.

```
sudo vi /etc/nginx/sites-available/example.com
```

Edit the VCL.

```
sub vcl_recv {
    unset req.http.cookie;
    if (req.url ~ “testhtml.htm”) {
        return (pass);
    }
}
```

<h2 id="config5">Config 5. Nginx w/FastCGI Cache

Base config, but FastCGI cache added to Nginx config.

```
sudo vi /etc/nginx/sites-available/default
sudo vi /etc/nginx/sites-available/example.com
```

Make changes to the root HTTP clock.

```
http {
    ...
    fastcgi_cache_path /path/to/cache  levels=1:2 keys_zone=[example]:10m inactive=5m;
    fastcgi_cache_key "$scheme$request_method$host$request_uri";
    ...
    server {
    ...
        location ~ \.php$ {
            ...
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_cache [example];
            fastcgi_cache_valid any 1m;
            fastcgi_cache_use_stale error timeout invalid_header http_500;
        }
    }
}
```

<h2 id="config6">Config 6. Part 1: Nginx using FastCGI w/ TCP/IP

Base config, but change communication to Unix sockets.

```
sudo vi /etc/php5/fpm/pool.d/www.conf
```

Edit the listen variable.

```
listen = 127.0.0.1:9000
```

Edit the Nginx config too.

```
sudo vi /etc/nginx/sites-available/example.com
```

Change the required FastCGI parameter.

```
fastcgi_pass 127.0.0.1:9000;
```

## Config 6. Part 2: Nginx using FastCGI w/ Unix Sockets

Base configuration.
