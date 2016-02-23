This is to setup virtual hosting, PHP, MySQL on ramdisk, and SSL on Mac OS X 10.10 (Yosemite).

<ol>
<li><a href="#step1">[View]</a> Step 1. Configure Virtual Hosting</li>
<li><a href="#step2">[View]</a> Step 2. Configure PHP</li>
<li><a href="#step3">[View]</a> Step 3. Configure SSL</li>
<li><a href="#step4">[View]</a> Step 4. Install MySQL onto a Ramdisk</li>
<li><a href="#step5">[View]</a> Step 5. Extras</li>
</ol>

<h2 id="step1">Step 1. Configure Virtual Hosting</h2>

Open the httpd config file:

    sudo vi /etc/apache2/httpd.conf

Uncomment these lines:

    LoadModule vhost_alias_module libexec/apache2/mod_vhost_alias.so
    ...
    LoadModule userdir_module libexec/apache2/mod_userdir.so
    ...
    Include /private/etc/apache2/extra/httpd-userdir.conf
    ...
    Include /private/etc/apache2/extra/httpd-vhosts.conf

Open the userdir file:

    sudo vi /private/etc/apache2/extra/httpd-userdir.conf

Uncomment these lines:

    Include /private/etc/apache2/users/*.conf

Create a file for user permissions:

    sudo vi /etc/apache2/users/[username].conf

Copy and paste the following lines into the file:

    <Directory "/path/to/web/folder">
    Options FollowSymLinks Indexes MultiViews
    Require all granted
    AllowOverride All
    </Directory>

Open the hosts file:

    sudo vi /etc/hosts

Add a record for the website:

    127.0.0.1 [example].localhost

Open the vhosts file:

    sudo vi /etc/apache2/extra/httpd-vhosts.conf

Copy, paste and configure the following lines:

    <VirtualHost *:80>
    DocumentRoot "/path/to/web/folder"
    ServerName [example].localhost
    </VirtualHost>

Restart Apache:

    sudo apachectl restart

Test the website:

    http://example.localhost

<h2 id="step2">Step 2. Configure PHP</h2>

Open the httpd config file:

    sudo vi /etc/apache2/httpd.conf

Enable PHP by uncommenting this line:

    LoadModule php5_module libexec/apache2/libphp5.so

Copy and open the PHP config file:

    sudo cp /etc/php.ini.default /etc/php.ini
    sudo vi /etc/php.ini

Set the timezone to UTC:

    date.timezone = UTC

Set owner permissions to cache and logs:

    cd /path/to/web/folder
    sudo rm -rf app/cache/*
    sudo rm -rf app/logs/*
    sudo chmod +a "www allow delete,write,append,file_inherit,directory_inherit" app/cache app/logs
    sudo chmod +a "`whoami` allow delete,write,append,file_inherit,directory_inherit" app/cache app/logs

Restart Apache:

    sudo apachectl restart

Test website:

    http://example.localhost/index.php

<h2 id="step3">Step 3. Configure SSL</h2>

Open the httpd config file:

    sudo vi /etc/apache2/httpd.conf

Enable HTTPS/SSL by uncommenting these lines:

    LoadModule ssl_module libexec/apache2/mod_ssl.so
    ...
    LoadModule socache_shmcb_module libexec/apache2/mod_socache_shmcb.so
    ...
    Include /private/etc/apache2/extra/httpd-ssl.conf

Create a certificate:

    cd /etc/apache2
    sudo ssh-keygen -f server.nopass.key
    sudo openssl req -new -key server.nopass.key -out request.csr
    sudo openssl x509 -req -days 365 -in request.csr -signkey server.nopass.key -out server.crt
    sudo openssl rsa -in server.nopass.key -out server.key

Open the SSL httpd config file:

    sudo vi /etc/apache2/extra/httpd-ssl.conf

Remove existing blocks, then add and configure these lines:

    DocumentRoot "/path/to/web/folder";
    ServerName [example].localhost

Restart Apache:

    sudo apachectl restart

Test the website:

    https://example.localhost

<h2 id="step4">Step 4. Install MySQL onto a Ramdisk</h2>

<ul>
<li>Download and install MySQL from the <a href="http://www.mysql.com/downloads/mysql/">download page</a>.</li>
<li>Make sure it's the 64-bit DMG archive version</li>
</ul>

Copy and open the MySQL config file:

    sudo cp /usr/local/mysql/support-files/my-default.cnf /etc/my.cnf
    sudo vi /etc/my.cnf

Enter the following config into my.cnf:

    [mysqld]
    ...
    default_time_zone = '+00:00'
    datadir = /Volumes/ramdisk/data
    innodb_flush_log_at_trx_commit = 2
    innodb_data_home_dir = /Volumes/ramdisk/data

Create a bash profile file to set the path and aliases:

    sudo vi ~/.bash_profile

Add these lines to the file:

    export PATH="/usr/local/mysql/bin:$PATH"
    alias ramdisk="/path/to/ramdisk.sh"
    alias mysetup="/path/to/mysetup.sh"

Reload your profile:

    su - [username]

Create and save a ramdisk.sh script file with the following lines:

    diskutil erasevolume HFS+ "ramdisk" `hdiutil attach -nomount ram://2048000`
    sudo cp -R /usr/local/mysql/data /Volumes/ramdisk/data

Create and save a mysetup.sh script file with the following lines:

    /usr/local/mysql/support-files/mysql.server start

    mysql -u root -e "create database if not exists [database];"
    mysql -u root -e "create user '[username]' identified by '[password]';"
    mysql -u root -e "grant all privileges on *.* to [database]@localhost;"
    mysql -u root -e "use mysql; update user set password=password('[password]') where user ='[username]';"
    mysql -u root -e "flush privileges;"

    /usr/local/mysql/support-files/mysql.server restart

Open the PHP config file:

    sudo vi /etc/php.ini

Find and update this line:

    pdo_mysql.default_socket=/tmp/mysql_ram.sock

Reset temporary admin password

    ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';

Run the scripts to create the ramdisk and setup the database:

    ramdisk
    mysetup

<h2 id="step5">Step 5. Extras</h2>

Install and configure git:

    git

Follow propmts to install command line tools, then configure git:

    git config --global user.name 'username'
    git config --global user.email 'email'

Save credentials in the keychain:

    git config --global credential.helper osxkeychain

Configure user to not require password:

    sudo chown [username] -R ~
    sudo chown [username] -R /path/to/websites
    sudo visudo

Then add this ine under the admin user:

    [username] ALL=(ALL) NOPASSWD: ALL

Install node.js and tools:

    http://nodejs.org
    sudo npm install -g gulp

Install Java for various tools:

    http://support.apple.com/kb/dl1572

Stop writing of Mac DS_Store files:

    defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool true
