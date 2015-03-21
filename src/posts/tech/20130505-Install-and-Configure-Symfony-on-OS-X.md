This is to setup Apache, MySQL and PHP for Symfony on Mac OS X. It includes setting up a virtual host, self-signed SSL certificate, git repository, and the database on ramdisk to increase PHPUnit speed.

## Configure Apache

Use vi to configure the httpd.conf file.

```
sudo vi /etc/apache2/httpd.conf
```

Enable PHP and virtual hosts by uncommenting these lines:

```
LoadModule php5_module libexec/apache2/libphp5.so
...
Include /private/etc/apache2/extra/httpd-vhosts.conf
...
Include /private/etc/apache2/extra/httpd-ssl.conf
```

Create a conf file to give your user permissions.

```
sudo vi /etc/apache2/users/[username].conf
```

Copy and paste the following into the new conf file.

```
&lt;Directory /&gt;
    Options FollowSymLinks Indexes MultiViews
    AllowOverride All
    Order allow,deny
    Allow from all
&lt;/Directory&gt;
```

## Install &amp; Configure Git

Download and install the latest version of Git for Mac <a href="http://git-scm.com">here</a>. Reboot after the install and then configure.

```
sudo reboot
git config --global user.name 'username'
git config --global user.email 'email'
```

Setup your password in the OS X keychain.

```
git credential-osxkeychain
git config --global credential.helper osxkeychain
```

## Configure Virtual Hosts

Use vi to configure the hosts file.

```
sudo vi /etc/hosts
```

Add a virtual hostname for each website.

```
127.0.0.1 [example].localhost
```

Configure directories and logs for apache.

```
sudo vi /etc/apache2/extra/httpd-vhosts.conf
```

Copy and past this block for each website.

```
&lt;VirtualHost *:80&gt;
    DocumentRoot '/path/to/web/folder'
    ServerName [example].localhost
&lt;/VirtualHost&gt;
```

## Setup Self-Signed SSL Certificate

When setting up the SSL certificate below, leave all questions blank.

```
cd /etc/apache2
sudo ssh-keygen -f server.key
sudo openssl req -new -key server.key -out request.csr
sudo openssl x509 -req -days 365 -in request.csr -signkey server.key -out server.crt
sudo openssl rsa -in server.key -out server.nopass.key
sudo vi /etc/apache2/extra/httpd-ssl.conf
```

Configure HTTPS virtual hosts.

```
DocumentRoot '/path/to/web/folder';
ServerName [example].localhost
...
SSLCertificateFile '/etc/apache2/server.crt';
SSLCertificateKeyFile '/etc/apache2/server.nopass.key';
```

## Install &amp; Configure MySQL

<ul>
<li>Download and install MySQL from the <a href="http://www.mysql.com/downloads/mysql/">download page</a>.</li>
<li>Make sure it's the 64-bit DMG archive version</li>
</ul>

Create config file from default.

```
sudo cp /usr/local/mysql/support-files/my-default.cnf /etc/my.cnf
sudo vi /etc/my.cnf
```

Enter the following config into my.cnf:

```
[mysqld]
...
default_time_zone = '+00:00'
datadir = /Volumes/ramdisk/data
innodb_flush_log_at_trx_commit = 2
innodb_data_home_dir = /Volumes/ramdisk/data
```

Create a bash profile to set the MySQL path and aliases.

```
sudo vi ~/.bash_profile
```

Add these lines to the file:

```
export PATH="/usr/local/mysql/bin:$PATH"
alias ramdisk_setup='/path/to/ramdisk_setup.sh'
alias mysql_setup='/path/to/mysql_setup.sh'
```

Here is the ramdisk_setup.sh script:

```
#!/bin/bash
/usr/local/mysql/bin/mysqladmin -u root shutdown
sudo diskutil umount force /Volumes/ramdisk
sudo umount -f /Volumes/ramdisk

diskname=`hdiutil attach -nomount ram://2048000`
newfs_hfs -v ramdisk /dev/disk1
sudo rm -R /Volumes/ramdisk
sudo mkdir /Volumes/ramdisk
sudo mount_hfs -u _mysql -g wheel -w -m 777 /dev/disk1 /Volumes/ramdisk
sudo cp -R /usr/local/mysql/data /Volumes/ramdisk/data

cd /Volumes/ramdisk
sudo chown -R _mysql:wheel /Volumes/ramdisk
sudo chmod -R 777 /Volumes/ramdisk
/usr/local/mysql/support-files/mysql.server start
sudo chown -R _mysql:wheel /Volumes/ramdisk
sudo chmod -R 777 /Volumes/ramdisk'
```

Here is the mysql_setup.sh script:

```
/usr/local/mysql/support-files/mysql.server start

mysql -u root -e "create database if not exists [database];"
mysql -u root -e "create user '[username]' identified by '[password]';"
mysql -u root -e "grant all privileges on *.* to [database]@localhost;"
mysql -u root -e "use mysql; update user set password=password('[password]') where user ='[username]';"
mysql -u root -e "flush privileges;"

/usr/local/mysql/support-files/mysql.server stop
/usr/local/mysql/support-files/mysql.server start
```

Reload your profile.

```
su - [username]
```

## Configure php.ini

Enable the php.ini file and open it for editing.

```
sudo cp /etc/php.ini.default /etc/php.ini
sudo vi /etc/php.ini
```

Change the timezone and unix socket path for MySQL to ramdisk.

```
date.timezone = UTC
...
pdo_mysql.default_socket=/tmp/mysql_ram.sock
...
detect_unicode = Off (at end, doesn't yet exist)
```

## Start Services

Restart Apache and use the pref pane to start MySQL.

```
sudo apachectl restart
```

## Optional Extras

Start visudo to change user settings.

```
sudo visudo
```

Add this line to remove the need to enter your password:

```
[username] ALL=(ALL) NOPASSWD: ALL
```

Install composer if necessary.

```
curl -s http://getcomposer.org/installer | php
php composer.phar install
```

Set permissions on cache and logs.

```
cd /path/to/web/folder
sudo rm -rf app/cache/*
sudo rm -rf app/logs/*
sudo chmod +a "www allow delete,write,append,file_inherit,directory_inherit" app/cache app/logs
sudo chmod +a "`whoami` allow delete,write,append,file_inherit,directory_inherit" app/cache app/logs
```

Install Java by initiating the installer.

```
java -version
```
