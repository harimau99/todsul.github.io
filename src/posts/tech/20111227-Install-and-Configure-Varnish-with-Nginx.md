Varnish is an HTTP accelerator (reverse proxy, load balancer, etc) with ESI capbilities. It serves cached content from memory, which helps to reduce load on web servers and databases. Let's install Varnish.

## Add the Repository

Change the version number to suit. The 3.0 path installs 3.0.1 at this time.

    sudo su -
    curl http://repo.varnish-cache.org/debian/GPG-key.txt | apt-key add -
    echo "deb http://repo.varnish-cache.org/ubuntu/ $(lsb_release -s -c) varnish-3.0" &gt;&gt; /etc/apt/sources.list

## Install the Package

The -V switch confirms the version.

    su [user]
    sudo aptitude update
    sudo aptitude install varnish
    varnishstat -V

## Configure the VCL

VCL is the Varnish Configuration Language. The VCL file holds most of the config.

    sudo vi /etc/varnish/default.vcl

Tell Varnish to communicate with the content server (Nginx) on port 8080.

    backend default {
        .host = "127.0.0.1";
        .port = "8080";
    }

With the above config, Varnish will only cache files without cookies. The following config strips all cookies and caches everything. The ideal config is somewhere in between.

    sub vcl_recv {
        unset req.http.cookie;
    }
    sub vcl_fetch {
        unset beresp.http.set-cookie;
    }

## Configure the Daemon

The Varnish daemon file sets ports, hosts, storage method, etc.

    sudo vi /etc/default/varnish

Tell Varnish to listen on port 80.

    DAEMON_OPTS="-a :80
                 -T localhost:6082
                 -f /etc/varnish/default.vcl
                 -S /etc/varnish/secret
                 -s malloc,256M"

## Configure Nginx Hosting

Varnish sits in front of Nginx on port 80 and talks to Nginx on 8080. Nginx defaults to 80, so make sure every Nginx server config listens on 8080. If not, Nginx will intercept Varnish.

    sudo vi /etc/nginx/sites-available/default
    sudo vi /etc/nginx/sites-available/example.com

Add the listen directive to the default Nginx server config.

    server {
        listen 8080;
        ...
    }

Do the same for the example.com virtual host config.

    index index.php;
    server {
        listen 8080;
        server_name www.example.com;
        ...
    }
    server {
        listen 8080;
        server_name example.com;
        ...
    }

## Restart Services

After restart, inspect the page headers for Via:1.1 varnish. The 1.1 refers to HTTP 1.1.

    sudo service nginx restart
    sudo service php5-fpm restart
    sudo service varnish restart

## Troubleshooting

If you see an Nginx error when you visit a page, or if page headers don't mention Varnish, it's likely Nginx is still listening on port 80. Turn off Varnish and use netstat to check ports.

    sudo /etc/init.d/varnish stop
    netstat -an | grep LISTEN

If you see anything on port 80, make sure all Nginx virtual hosts are listening on port 8080.
