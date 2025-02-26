# MONOS
*See for free*

**M O N**itoring **O**pen-source **S**ystem


*MONOS* is a free software which allows you to monitor devices in your network. Monitored devices are shown on a web application.

# MANUAL FOR SNMP MONITORING

<details>
  <summary>SNMP SERVER</summary>
  <a href="#snmp-server">SNMP Server</a>  |  <a href="#snmp-client">SNMP Client</a>
</details>
<details>
  <summary>WEB SERVER, PHP SERVER</summary>
  <a href="#apache">Apache Server</a>  |  <a href="#php">PHP Server</a>
</details>
<details>
  <summary>DATABASE</summary>
  <a href="#db">MariaDB</a>  |  <a href="#db-setup">Database Set Up</a>
</details>

## <a name="snmp-server"> SNMP SERVER: </a>

### Install SNMP packages:
```sh
sudo dnf install net-snmp net-snmp-utils
```

### Edit the configuration file:
```sh
sudo nano /etc/snmp/snmpd.conf
```

### Example configuration:
```sh
# Update [COMMUNITY] with your preferred string
rwcommunity [COMMUNITY] default

# Disk monitoring
disk  / 100

# Agent user
agentuser  [USER]

# Agent address
agentAddress udp:161

# System location and contact
syslocation Unknown
syscontact Root <root@localhost>

# Access control
access  [COMMUNITY] "" any noauth exact systemview none none

# Logging
dontLogTCPWrappersConnects yes
```

### Start and enable the SNMP service:
```sh
sudo systemctl enable snmpd
sudo systemctl start snmpd
```

### Test the SNMP configuration:
```sh
snmpwalk -v2c -c [COMMUNITY] localhost
```

## <a name="snmp-client"> SNMP CLIENT: </a>

### Install SNMP packages:
```sh
sudo apt update
sudo apt install snmpd
```


### Edit the SNMP configuration file:
```sh
sudo nano /etc/snmp/snmpd.conf
```



### Example configuration:
```sh
# Update [COMMUNITY] with your preferred string
rocommunity [COMMUNITY] default

# Disk monitoring
disk  / 100

# System location and contact
syslocation Home
syscontact Admin <admin@localhost>

# Access control
access  [COMMUNITY] "" any noauth exact systemview none none

# Agent address
agentAddress udp:161,udp6:[::1]:161
```


### Edit the default SNMP settings:
```sh
sudo nano /etc/default/snmpd
```

**Change the line:**
```sh
SNMPDOPTS='-Lsd -Lf /dev/null -p /run/snmpd.pid -a'
```

**TO:**
```sh
SNMPDOPTS='-Lsd -Lf /dev/null -p /run/snmpd.pid -a -x tcp:localhost:161'
```


### Restart the SNMP service:
```sh
sudo systemctl restart snmpd
```


### Test the SNMP configuration:
```sh
snmpwalk -v2c -c [COMMUNITY] localhost
```


## <a name="apache"> WEB SERVER & PHP SERVER </a>


### 1. Update System Packages
First, ensure your system packages are up to date:
```sh
sudo dnf update
```

### 2. Install Apache
Apache is the web server that will serve your PHP files.
```sh
sudo dnf install httpd
sudo systemctl enable httpd
sudo systemctl start httpd
```

### 3. Install PHP
Next, install PHP and the necessary modules:
```sh
sudo dnf install php php-common php-mysqlnd php-snmp #php-gd php-xml php-mbstring php-json php-curl
```

### 4. Configure Firewall
Allow HTTP and HTTPS traffic through the firewall:
```sh
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

### 5. Create a PHP File
Create an index.php file in the web server’s root directory:
```sh
sudo nano /var/www/html/index.php
```


### Open the Apache Configuration File:
```sh
sudo nano /etc/httpd/conf/httpd.conf
```


### Add the DirectoryIndex Directive:
Locate the DirectoryIndex directive and modify it as follows:
```sh
DirectoryIndex index.php index.html
```


### File Permissions
Set file permissions for /html to ensure all files and directories are accessible.
```sh
sudo chmod -R 755 /var/www/html
```


### Configure Apache to Use PHP
Ensure that Apache is configured to handle PHP files. Open the Apache configuration file:
```sh
sudo nano /etc/httpd/conf/httpd.conf
```


### Add or ensure the following lines are present:
```sh
#LoadModule php_module modules/libphp.so
AddHandler php-script .php
```


Add these lines to display php files properly:
```sh
<FilesMatch \.php$>
    SetHandler application/x-httpd-php
</FilesMatch>
```


### Use PHP-FPM (Optional)
If problems appear this might help.
```sh
sudo systemctl enable php-fmp
sudo systemctl start php-fpm
```


### Restart Apache:
Restart the Apache server to apply the changes.
```sh
sudo systemctl restart httpd
```


Download MONOS application using curl or wget:
```sh
curl https://github.com/MatejPiskac/MONOS/versions/monos.tar.gz -o - | tar -xz -C /var/www/html/monos
```



## <a name="php"> PHP AND SNMP </a>

### Enable SNMP in PHP

First look for php.ini file, it should be in /etc/php.ini or you can look for it using this command:
```sh
php –ini
```


Now enable SNMP in PHP by editing the php.ini file. Add the following line:
```sh
extension=snmp
```


### Restart Apache:
Restart the Apache server to apply the changes.
```sh
sudo systemctl restart httpd
```


## <a name="db"> DATABASE SETUP </a>

### Install MariaDB
```sh
sudo dnf install mariadb-server php-mysqli
sudo systemctl enable mariadb
sudo systemctl start mariadb
```

### <a name="db-setup"> Download DB setup script </a>
Visit [debstream.org](www.debstream.org/monos/db-setup.sh) and download _db-setup.sh_
```sh
curl https://www.debstream.org/monos/db-setup.sh /your/path/
```

Run _db-setup.sh_ on your server:
```sh
sudo chmod +x db-setup.sh
sudo bash db-setup.sh
```

### Grant permissions to user
Enter MariaDB:
```sh
ALTER USER 'your_username'@'localhost' IDENTIFIED BY 'new_password';
```

Reset password for user (if unsure):
```sh
ALTER USER 'your_username'@'localhost' IDENTIFIED BY 'new_password';
```

Flush privilages:
```sh
FLUSH PRIVILEGES;
```

Restart Apache and MariaDB:
```sh
sudo systemctl restart httpd
sudo systemctl restart mariadb
```


## Setup Debian Server for MONOS

### Install required dependencies
```sh
sudo apt install -y snmp snmpd libsnmp-dev snmp-mibs-downloader php-snmp php php-mysqli apache2 libapache2-mod-php mariadb-server 
```

### Install MIBs for SNMP
```sh
sudo download-mibs
```

### Edit configuration of SNMP (snmpd.conf)
```sh
nano /etc/snmp/snmpd.conf
```
Content:
```sh
rwcommunity [COMMUNITY] default

# Disk monitoring
disk  / 100

# Agent user
agentuser  [USER]

# Agent address
agentAddress udp:161

# System location and contact
syslocation Unknown
syscontact Root <root@localhost>

# Access control
access [COMMUNITY] "" any noauth exact systemview none none

# Logging
dontLogTCPWrappersConnects yes
```

### Enable `mysqli` extension
Locate `php.ini` file
```sh
find / | grep php.ini
```
Edit the file
```sh
nano /etc/php/<version>/apache2/php.ini
```
Enable the extension by adding or uncommenting:
```sh
extension=mysqli
```

### Install MONOS Aplication
Navigate to `/var/www/html/` directory:
```sh
cd /var/www/html/
```
Download the Monos App using `wget` or `git`
```sh
wget https://monos.debstream.org/app/download
```
```sh
git clone https://github.com/DebStream-Solutions/monos.git
# git clone https://username:<pat>@github.com/<your account or organization>/<repo>.git
```

### Configure Monos database

Navigate to directory MONOS
```sh
cd /var/www/html/MONOS
```

Run `db-setup.sh` script to configure database
```sh
sudo /.db-setup.sh
```

Set a strong password for the database and admin user
1. Minimum `8 characters` long
2. Contains atleast `1 number`
3. Contains atleast `1 lowercase` and `1 uppercase` letter
4. Contains atleast `1 special character`

| Login to Monos with username `admin` and the admin password <br>
| Login to database with username `mroot` and the admin password


Restart services
```sh
sudo systemctl restart apache2
sudo systemctl restart mariadb
sudo systemctl restart snmpd
```

Now everything is set up on your server and you can prepare the clients


## Client setup - Server & Workstation

