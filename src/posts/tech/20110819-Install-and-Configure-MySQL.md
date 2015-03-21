This shows how to install and configure MySQL server on a Linux server. It includes setting up users andprivileges, moving databases between servers, and imposing UTF-8 across the board.

## Install MySQL and PHP5 support

```
sudo aptitude update
sudo aptitude install mysql-server
sudo aptitude install php5-mysql
```

## Secure MySQL and Login

Choose yes for all questions, except to change the root password.

```
sudo mysql_secure_installation
mysql -u root -p
```

If necessary, add the mysql path (/usr/local/mysql/bin) to the profile and refresh.

```
cd
vi  ~/.profile
. ./.profile
echo $PATH
```

## Create a database

Change the root password, create database and users, give privileges.

```
mysqladmin -u root password [newpassword]
mysql -u root -p [password]
create database mytest;
create user 'username' identified by 'password';
grant all privileges on *.* to username@localhost;
flush privileges;
exit
```

## Export a Database

Switch to the superuser to avoid any permissions issues.

```
sudo su -
mysqldump -u [user] -p [database] &gt; [filename].sql
```

## Import a Database

For smaller files, import from the unix shell.

```
mysql -u [user] -p [database] &lt; [filename].sql
```

For larger files, use the source command inside mysql.

```
mysql -u [user] -p
use [database]
source &lt; [file].sql
```

## Set Charset

We want end-to-end UTF8.

```
ALTER DATABASE DEFAULT CHARACTER SET utf8;
ALTER TABLE [exampletable] DEFAULT CHARACTER SET utf8;
```
