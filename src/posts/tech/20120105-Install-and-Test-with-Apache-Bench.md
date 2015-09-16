Apache Bench is an easy-to-use tool for load testing and is available in most Linux distros. This article shows how to setup three tests: a static file test, a dynamic file (PHP) test, and a database (MySQL) access test.

Apache Bench is a load testing tool for web servers.

## Install Apache Bench

    sudo aptitude install apache2-utils

## Run Apache Bench

The following line uses keepalive for 50,000 connections, at a concurrency of 100 and a timeout of 20 seconds. Be sure to end root domains with a slash.

    ab -k -n 50000 -c 100 -t 20 http://example.com/

## Create a Static HTML Test File

    sudo vi /srv/www/example.com/web/testhtml.htm

Add the following content to the test file:

    <!DOCTYPE HTML>
    <html>
    <head><title>#1 HTML Test</title></head>
    <body>
    This is a web server test page.
    </body>
    </html>

## Create a Dynamic PHP File

    sudo vi /srv/www/example.com/web/testhtml.htm

Add the following content to the test file:

    <!DOCTYPE HTML>
    <?php echo '<html>'; ?>
    <?php echo '<head><title>#2 PHP Test</title></head>'; ?>
    <?php echo '<body>'; ?>
    <?php echo 'This is a web server test page.'; ?>
    <?php echo '</body>'; ?>
    <?php echo '</html>'; ?>

## Create a Dynamic MySQL File

    sudo vi /srv/www/example.com/web/testmysql.php

Add the following content to the test file:

    <!DOCTYPE HTML>
    <?php
       $link = mysql_connect("localhost", "username", "password");
       mysql_select_db("mytest");

       $query = "SELECT * FROM mytest";
       $result = mysql_query($query);

       while ($line = mysql_fetch_array($result))
       {
           print "$line[0]\n";
       }
       mysql_close($link);
    ?>

Configure the database for the test file:

    USE mytest
    CREATE TABLE mytest (content text) ENGINE=InnoDB AUTO_INCREMENT=11 DEFAULT CHARSET=latin1;
    INSERT INTO mytest (content) VALUES('<html><head><title>#1 HTML Test</title></head><body>This is a web server test page.</body></html>');
