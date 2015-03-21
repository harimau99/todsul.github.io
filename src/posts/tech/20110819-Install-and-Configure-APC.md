The Alternative PHP Cache (APC) is a free and open opcode cache for PHP. While it works out-of-the-box, it's important to configure it correctly. This shows how to setup APC and increase its efficiency.

## Install the Package

Use a package to install the latest version.

    sudo add-apt-repository ppa:brianmercer/apc
    sudo aptitude update
    sudo aptitude install php-apc

## Configure APC

    sudo vi /etc/php5/conf.d/apc.ini

The shm_size setting suggests APC uses shared kernel memory, but most APC installs are complied with MMAP support. This is why we can increase shm_size above our kernel shm_size. With MMAP, APC only uses one shm_segment.

    extension=apc.so
    apc.enabled=1
    apc.shm_segments=1
    apc.shm_size=64M
    apc.ttl=7200
    apc.user_ttl=7200
    apc.filters=apc\.php$

Increase shm_size to provide ample caching space and set TTLs so garbage collection works more efficiently. Also add a filter to bypass the APC stats page.

## Activate APC Stats

APC comes with a nifty stats page. Copy it to the web folder and unzip.

    sudo cp /usr/share/doc/php-apc/apc.php.gz /srv/www/example.com/web
    sudo gzip -d /srv/www/example.com/web/apc.php.gz
    sudo vi /srv/www/example.com/web/apc.php

Change the password to enable web-based purging and advanced stats.

    defaults('ADMIN_USERNAME','apc');
    defaults('ADMIN_PASSWORD','newpassword');

Reboot the server for APC to start and then test the page at http://example.com/apc.php.

## Monitor APC Caching

Keep an eye on the 'cache full count' value. This is a count of the number of times APC runs out of memory and has to remove stale objects or purge memory completely. Also watch the fragmentation level. Now test using Apache Bench.
