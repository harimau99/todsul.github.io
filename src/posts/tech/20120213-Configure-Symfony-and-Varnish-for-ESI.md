Edge-Side Includes allow you to cache dynamic content with various timeouts per segment. Varnish supports ESI out of the box. Let's look at how to configure the two for an optimal partial-caching strategy.

## Deploy VEMAP Stack

Install Varnish, Nginx, MySQL, APC and PHP.

## Move Files and Database

Rather than FTP'ing files individually, archive all prod files, upload the archive and unzip insitu. Once unzipped, set write permissions for the cache and logs directories.

```
cd /srv/www/example.com
sudo unzip Archive.zip
sudo chown [user] /srv/www/example.com
sudo chmod -R 777 /srv/www/example.com/app/cache
sudo chmod -R 777 /srv/www/example.com/app/logs
```

## Setup Resources

The Assetic asset dump requires Java. Without Java installed, run the dump locally and upload the files. The asset:install works fine though.

```
cd /srv/www/example.com
app/console assetic:dump
app/console assets:install --symlink web
```

## Configure Nginx for Symfony

At a minimum, the config should achieve the following:

<ul>
<li>Send requests to the Symfony front controller</li>
<li>Abstract the 8080 port for the application and user</li>
<li>Turn off access logging for better performance</li>
<li>Rewrite trailing slash for URL canonicalization</li>
<li>Use Unix sockets for better performance (debateable)</li>
<li>Set Cache Control headers on static files</li>
</ul>


Open the Nginx config.

```
sudo vi /etc/nginx/sites-available/example.com
```

Copy and paste the following:

```
index app.php;
access_log off;
server {
    listen 8080;
    server_name www.example.com;
    rewrite ^ $scheme://example.com$request_uri? permanent;
}
server {
    listen 8080;
    server_name example.com;
    error_log /srv/www/example.com/logs/error.log;
    rewrite ^/(.*)/$ /$1 permanent;
    root /srv/www/example.com/web;
    location / {
        try_files $uri $uri/ /app.php;
    }
    location ~* \.(?:ico|css|js|gif|jpe?g|png)$ {
        expires max;
        add_header Cache-Control "public, must-revalidate, proxy-revalidate";
    }
    location ~ \.php$ {
        try_files $uri = 404;
        include fastcgi_params;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_param  SERVER_PORT 80;
    }
}
```

## Configure Varnish for Symfony

Remove any cookie manipulation and add ESI handling.

```
sudo vi /etc/varnish/default.vcl
```

Copy and paste the following:

```
sub vcl_recv {
    set req.http.Surrogate-Capability = "abc=ESI/1.0";
}
sub vcl_fetch {
    if (beresp.http.Surrogate-Control ~ "ESI/1.0") {
        unset beresp.http.Surrogate-Control;
        set beresp.do_esi = true;
    }
}
```

## Configure Symfony for ESI

See the Symfony2 ESI cookbook for more info. Start by turning on ESI in config.yml.

```
framework:
    esi:             true
```

Turn on Assetic asset versioning for cache bursting. This is essential for the aggressive (expires max;) asset caching in the Nginx config. For assets not handled by Assetic (e.g. images inside CSS files) add versioning manually (e.g. logo.png?v=1).

```
framework:
    templating:      { engines: ['twig'], assets_version: v1 }
```

Set sections of the website as standalone (ESI enabled). This allows various TTLs on a single page. Varnish puts each section together to construct the page before caching.

```
{% render 'Vendor:Bundle:controller' with {}, {'standalone': true} %} // or...
$this-&gt;render('Vendor:Example:temp.html.twig', array('standalone' =&gt; true));
```

Set shared max age for responses and sections. If you set the max age (non-shared), ESI will not work because the whole page is cached.

```
$response-&gt;setSharedMaxAge(60);
```

## Setup Authentication

Since Symfony is running on Nginx on port 8080, make these adjustments to support Symfony authentication. Firstly, in AppKernal.php, add this line before $kernel->handle():

```
Request::trustProxyData();
```

Also, add these changes to the Varnish VCL file:

```
set req.http.X-Forwarded-Port = "80";
if (req.http.Authorization || req.http.Cookie) {
    return (pass);
}
```

## Test Setup

Clear all caches, restart services and test.

```
sudo app/console cache:clear --env=prod --no-warmup
sudo service nginx restart
sudo service php5-fpm restart
sudo service varnish restart
```
