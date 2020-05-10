# LinuxLoka
Lokaverkefni KEST2NLI05EU

## Static IP
Add static IP entries to the following file:
```sudo nano /etc/netplan/00-installer-config.yaml```

## DHCP

```sudo apt-get install isc-dhcp-server -y```
```sudo nano /etc/dhcp/dhcpd.conf```

```sudo service isc-dhcp-server-restart```

## DNS
    1. sudo apt install bind9 bind9utils bind9-doc bind9-host
    2. names -v
    3. named -v
    4. systemctl status bind9
    5. sudo systemctl enable bind9
    6. sudo netstat -lnptu | grep named
    7. sudo rndc status
    8. sudo nano /etc/bind/named.conf.options
    9. sudo named-checkconf
    
## User creation script
```bash
#! /usr/bin/env bash
#set -x

MY_INPUT='/root/temp/input'
declare -a A_NAME
declare -a A_FNAME
declare -a A_LNAME
declare -a A_UNAME
declare -a A_MAIL
declare -a A_DEP
declare -a A_EMPID
declare -a A_PASS

A_PASS = "pass.123"
while IFS=, read -r COL1 COL2 COL3 COL4 COL5 COL6 COL7 TRASH; do
    A_NAME +=("COL1")
    A_FNAME +=("COL2")
    A_LNAME +=("COL3")
    A_UNAME +=("COL4")
    A_MAIL +=("COL5")
    A_DEP +=("COL6")
    A_EMPID +=("COL7")
done <"$MY_INPUT"

for index in "${!A_UNAME[@]}"; do
    useradd -g "${A_DEP[$index]}" -d "/home/${A_UNAME[$index]}" -s /bin/bash -p "$(echo "${A_PASS}" | openssl passwd -1 -stdin)" "${A_UNAME[$index]}" -u "${A_EMPID[$index]}"
done
```

# SQL
Install SQL server
```
$ sudo apt install mysql-server
```
And then secure it
```
$ sudo mysql_secure_installation
```
After securing you will need to restart mysql and then enable it for it to take effect
```
$ sudo systemctl restart mysql
$ sudo systemctl enable mysql
```
By default, MySql listens for connections on port 3306. You can confirm that your MySql service is listening with this command:
```
$ ss -ltn
```
The only thing left is to make sure your firewall is nott blocking incoming connections on port 3306. You can issue the following command to add an exception in Ubuntu's default firewall:
```
$ sudo ufw allow from any to any port 3306 proto tcp
```
####Setting up
To open up MySql execute the following command:
```
$ sudo mysql
```
To create a new databes issue this command but remember to replace "my_database" with the desired name of your database
```
mysql> CREATE DATABASE my_database;
```
Next you will nerd to create a user account that will have privileges. This will create a user named "my_user" with the password "my_password". The user can connect from anywhere on the internet with the '%', you can restrict this to an IP address or to a "localhost"
```
mysql> CREATE USER 'my_user'@'%' IDENTIFIED BY 'my_password';
```
To grant the user all permmissions on the database issue the following command:
```
mysql> GRANT ALL PRIVILEGES ON my_database.* to my_user@'%';
```
Finally, save all the changes with this command and then use the "exit" command to close the MySql terminal
```
mysql> FLUSH PRIVILEGES;
mysql> exit
```
