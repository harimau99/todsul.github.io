This is a basic low-security setup guide of Ubuntu Server for web hosting. It includes creating a user account, setting up IP Tables, configuring the timezone, and running a full update.

## Remove Old Keys

This is required if the client computer has keys linked to a previous build. Find the IP address in the Linode control panel under the remote access tab and run the following command.

    ssh-keygen -R [server_ip]

## Login using SSH

Remote into the node using the root password you chose when rebuilding the node.

    ssh root@[server_ip]

## Configure Hostname

Use vi to edit files. Search Google to find basic commands.

    echo "[hostname]" &gt; /etc/hostname
    hostname -F /etc/hostname
    vi /etc/hosts

Add the second line to your hosts files.

    127.0.0.1 localhost.localdomain localhost
    [server_ip] [hostname].example.com [hostname]

## Create User Account

Create a user and give it ALL permissions.

    /usr/sbin/adduser [username]
    visudo

Add this line to the file.

    [username] ALL=(ALL) ALL

## Configure SSH over PKA

Generate a local public key and push it to the server.

    ssh-keygen
    scp ~/.ssh/id_rsa.pub root@example.com:/home/[user]

Copy the key to .ssh and rename to authorized_keys

    cd ~/
    mkdir .ssh
    sudo mv id_rsa.pub .ssh/authorized_keys

## Setup Firewall

Use iptables as a firewall to manage port access and store rules in a file.

    sudo /sbin/iptables -F
    sudo vi /etc/iptables.up.rules

These rules allow loopback, established, outbound, HTTP, HTTPS, SSH and ping traffic.

    * filter
    -A INPUT -i lo -j ACCEPT
    -A INPUT ! -i lo -d 127.0.0.0/8 -j REJECT
    -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
    -A OUTPUT -j ACCEPT
    -A INPUT -p tcp --dport 80 -j ACCEPT
    -A INPUT -p tcp --dport 443 -j ACCEPT
    -A INPUT -p tcp -m state --state NEW --dport 22 -j ACCEPT
    -A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
    -A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables denied: " --log-level 7
    -A INPUT -j REJECT
    -A FORWARD -j REJECT
    COMMIT

Import the settings file and create the script to re-import the rules on restart.

    sudo /sbin/iptables-restore &lt; /etc/iptables.up.rules
    sudo /sbin/iptables -L
    sudo vi /etc/network/if-pre-up.d/iptables

Add these lines to the script file.

    #!/bin/sh
    /sbin/iptables-restore &lt; /etc/iptables.up.rules

Add execute permissions to the script, reboot, then login over SSH as our new user to test the firewall. Start commands with sudo when you login as the new user.

    sudo chmod +x /etc/network/if-pre-up.d/iptables
    sudo reboot
    ssh [username]@[server_ip]
    sudo /sbin/iptables -L

## Set Timezone

Set the timezone to where most users reside. UTC is a good choice.

    sudo dpkg-reconfigure tzdata

## Update and Upgrade

As a new build, update our repository source list and run any necessary upgrades.

    sudo aptitude update
    sudo aptitude full-upgrade
