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
## NTP
### Server setup
Install the package for ntp
```bash
sudo apt install ntp
```

configure the ntp server

```bash
sudo nano /etc/ntp.conf
```

add ntp servers such as 
```bash
0.us.pool.ntp.org
```
or a similar server for your locale

restart the ntp server
```bash
sudo systemctl restart ntp
```

### client setup
Install packages
```bash
sudo apt install ntpdate
```

Set the ntp server connection
```bash
sudo ntpdate "10.10.0.14"
```
Disable default time sync
```bash
sudo timedatectl set-ntp off
```

Install ntp daemon
```bash
sudo apt install ntp
```

Add the following line to the configuration file: /etc/ntp.conf
```bash
server 10.10.0.14
```
restart ntp client
```bash
sudo systemctl restart ntp
```

Check the ntp configuration queue
```bash
ntpq -p
```

# Syslog
Install the syslog package
```bash
sudo apt install rsyslog -y
```

Open the configuration file for syslog
```bash
nano /etc/rsyslog.conf
```

uncomment the following lines
```
$ModLoad imudp
$UDPServerRun 514
$ModLoad imtcp
$InputTCPServerRun 514
```

define subnet, ip and domain of accessible computers

```
$AllowedSender TCP, 127.0.0.1, 10.10.0.0/28, *.ddp.local
$AllowedSender UDP, 127.0.0.1, 10.10.0.0/28, *.ddp.local
```

Tell syslog where to store log files

```
$template remote-incoming-logs, "/var/log/%HOSTNAME%/%PROGRAMNAME%.log" 
*.* ?remote-incoming-logs

```

Check for syntax errors in configuration file
```bash
rsyslogd -f /etc/rsyslog.conf -N1
```

### Configure client
open configuration file
```
nano /etc/rsyslog.conf
```

Add this to end of file
```
##Enable sending of logs over UDP add the following line:

*.* @10.10.0.14:514


##Enable sending of logs over TCP add the following line:

*.* @@10.10.0.14:514

##Set disk queue when rsyslog server will be down:

$ActionQueueFileName queue
$ActionQueueMaxDiskSpace 1g
$ActionQueueSaveOnShutdown on
$ActionQueueType LinkedList
$ActionResumeRetryCount -1
```

restart syslog on client

```
systemctl restart rsyslog
```

# Postfix mail server
Install postfix
```
sudo apt install postfix
```

Go through configuration wizard on screen

### Map mail addresses to linux accounts

Open config file
```bash
sudo nano /etc/postfix/virtual
```
add email entries for users here
```
amf@ddp.is AndFri
stg@ddp.is SteGis
```
Apply the mapping
```bash
sudo postmap /etc/postfix/virtual
```
restart postfix
```bash
sudo systemctl restart postfix
```

### Adjust firewall

Allow postfix connections through ufw firewall
```bash
sudo ufw allow postfix
```

### Setup mail client

install snail for managing mail
```bash
sudo apt install s-nail
```
edit snail settings file
```bash
sudo nano /etc/s-nail.rc
```
add the following to the file
```
set emptystart
set folder=Maildir
set record=+sent
```

