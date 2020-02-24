# vtiger-ubuntuserver-doc
Steps dercribing how to install & upgrade vTiger on Ubuntu Server 18.04

## Upgrading Vtiger 6.3 to 7.1 Directly with a clean installation

Initial reference guide: <https://www.vgsglobal.com/blog/upgrade-vtiger-6-to-7/>

1. Download and install a fresh copy of Vtiger 7.1 – Follow this tutorial if you need help with it (See Below)
2. In your old Vtiger disable all the custom modules and log out, do NOT close the tab.
3. Create a copy of the old database (See Below)
4. Edit config.inc.php and replace the database base name, user and password to connect to the database you create in step #3 (See Below)
5. Edit vtigerversion.php and replace 7.1.0 by your current vtiger version
6. Copy folders to the new installation (See Below)

7. Through browser open /migrate path : <http://192.168.1.178/index.php?module=Migration&view=Index&mode=step1> or <http://192.168.1.241/migrate/> (See Below)

   - If wanting to reset user in the DB: admin password to admin
update vtiger_users set user_password = 'adpexzg3FUZAk', crypt_type = '' where user_name = 'admin';

   - Check below for the ERROR: ERROR EXTRACTING MIGRATION ZIP FILE!

8. Edit vtigerversion.php to update the version 7.1.0
cd /var/www/html/vtigercrm
sudo nano vtigerversion.php
9. Re-install the custom modules with the zip file provided by your vendor if you need to.

### Fresh install vTiger: Download and install a fresh copy of Vtiger 7.1

Download & Install Ubuntu 18.04 Server

AWS: vTiger 7.1
Username: ubuntu
Password: NO PASSWORD, Public Key only

Local Server Name: vtiger
Username:
Password:

```sh
ifconfig
sudo apt-get update && sudo apt-get upgrade -y
# Install LAMP server
sudo apt-get install apache2 mariadb-server libapache2-mod-php7.2 php7.2 php7.2-cli php7.2-mysql php7.2-common php7.2-zip php7.2-mbstring php7.2-xmlrpc php7.2-curl php7.2-soap php7.2-gd php7.2-xml php7.2-intl php7.2-ldap php7.2-imap unzip wget -y

sudo nano /etc/php/7.2/apache2/php.ini
# or copy out & upload: sudo mv php.ini /etc/php/7.2/apache2/
```

- File php.ini changes

file_uploads = On
allow_url_fopen = On
memory_limit = 256M
upload_max_filesize = 30M
post_max_size = 40M
max_execution_time = 60
max_input_vars = 1500

```sh
# copy php.ini file back to server

sudo systemctl start apache2
sudo systemctl start mariadb
sudo systemctl enable apache2
sudo systemctl enable mariadb
```

- Configure DB

```sh
sudo mysql_secure_installation

	Enter current password for root (enter for none):
	Set root password? [Y/n]: N
	Remove anonymous users? [Y/n]: Y
	Disallow root login remotely? [Y/n]: Y
	Remove test database and access to it? [Y/n]:  Y
	Reload privilege tables now? [Y/n]:  Y

sudo mysql
MariaDB [(none)]> CREATE DATABASE vtigerdb;
MariaDB [(none)]> CREATE USER 'vtiger'@'localhost' IDENTIFIED BY 'password';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON vtigerdb.* TO 'vtiger'@'localhost' WITH GRANT OPTION;

MariaDB [(none)]> ALTER DATABASE vtigerdb CHARACTER SET utf8 COLLATE utf8_general_ci;

MariaDB [(none)]> FLUSH PRIVILEGES;
MariaDB [(none)]> exit
```

### Adminer or PHPMyAdmin install

```sh
# Install PHP Myadmin
sudo apt install phpmyadmin php-mbstring php-gettext -y
# setup password & apache2, password: JVTech default # in Testing: password
sudo phpenmod mbstring
sudo systemctl restart apache2
# Create a user for PHPmyadmin use
sudo mysql
CREATE USER 'myadmin'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'myadmin'@'localhost' WITH GRANT OPTION;
# Check users
select user, password, host, plugin from mysql.user;

exit
```

```sh
# Or Adminer (personal preference)
sudo apt install adminer
cd /usr/share/adminer
sudo php compile.php
sudo echo "Alias /adminer.php /usr/share/adminer/adminer-4.6.2.php" | sudo tee /etc/apache2/conf-available/adminer.conf
cd /etc/apache2/conf-available/
sudo a2enconf adminer.conf
sudo systemctl reload apache2

# Create a user for Adminer use
sudo mysql
CREATE USER 'adminer'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'adminer'@'localhost' WITH GRANT OPTION;
# Check users
select user, password, host, plugin from mysql.user;

exit
```

- To change password in MariaDB `SET PASSWORD FOR 'vtiger'@'localhost' = PASSWORD('password');`

```sh
# Install vtiger
wget https://excellmedia.dl.sourceforge.net/project/vtigercrm/vtiger%20CRM%207.1.0/Core%20Product/vtigercrm7.1.0.tar.gz
tar -xvzf vtigercrm7.1.0.tar.gz

sudo cp -r vtigercrm /var/www/html/
sudo chown -R www-data:www-data /var/www/html/vtigercrm
sudo chmod -R 755 /var/www/html/vtigercrm
```

```sh
# Configure Apache for vTiger CRM
sudo nano /etc/apache2/sites-available/vtigercrm.conf
```

<VirtualHost *:80>
     ServerAdmin example@example.com
     ServerName unitrading.tklapp.com
     DocumentRoot /var/www/html/vtigercrm/

     <Directory /var/www/html/vtigercrm/>
        Options FollowSymlinks
        AllowOverride All
        Require all granted
     </Directory>

     ErrorLog /var/log/apache2/vtigercrm_error.log
     CustomLog /var/log/apache2/vtigercrm_access.log combined
</VirtualHost>

```sh
sudo a2ensite vtigercrm
sudo a2dissite 000-default

sudo a2enmod rewrite
sudo systemctl restart apache2
systemctl status apache2
# Make sure apache is running properly
```

Go to web URL: 192.168.x.x or example.com

DB Information:
Host name: localhost  
User Name: vtiger
Password: password
DB Name: vtigerdb

Currency: HKD # or: USD
Admin user: admin
Password:
Email: a@xxx.com
Date Format: dd-mm-yyyy
Timezone: +8

- Follow on screen instructions to finish installation
- Choose to use modules: Contact Management, Invoicing & Inventory Management. For JV Tech add: Support.

### Create a copy of the old database

- Backup vTiger Directory & DB

Login to remote server (ex: root@unitrading.tklapp.com )

```sh
cd /var/www
sudo tar -zcvf vtigercrm.tar.gz vtigercrm

```

- Copy file out: vtigercrm.tar.gz

Backup vtiger DB use Adminer Dump DB: vtigercrm
Output: gzip
Format: SQL
Tables: DROP+Create
Data: INSERT

### Upload copy of DB to server via Adminer

- Browser point to: http/url/phpmyadmin/ ; user: myadmin; password: password
- Create New DB: vtigerold ; Collation: utf8_general_ci ;
- Import DB SQL file

```sh
sudo mysql
MariaDB [(none)]> CREATE DATABASE vtigerold;
MariaDB [(none)]> ALTER DATABASE vtigerold CHARACTER SET utf8 COLLATE utf8_general_ci;

MariaDB [(none)]> GRANT ALL PRIVILEGES ON vtigerold.* TO 'vtiger'@'localhost' WITH GRANT OPTION;

MariaDB [(none)]> FLUSH PRIVILEGES;
MariaDB [(none)]> exit
```

### Edit config.inc.php and replace the database base name, etc

```sh
cd /var/www/html/vtigercrm/
sudo nano config.inc.php
sudo nano vtigerversion.php
```

Change:
DB name: vtigerold
User: vtiger
Password: password

Change version to: '6.3.0' (or whatever the old version is)

### Copy folders to the new installation

Copy your custom module, storage & user_privileges folders to the new installation. 

For example Folder to Copy: PDFMaker.
Assume upload to directory: `~/vtigerlive/`

```sh
# Upload folder to ~/vtigerlive/modules/
cd && mkdir vtigerlive
cd ~/vtigerlive/modules/
sudo cp -r PDFMaker /var/www/html/vtigercrm/modules/

# Upload folder to ~/vtigerlive/storage/
cd ~/vtigerlive/storage/
sudo cp -r * /var/www/html/vtigercrm/storage/

# Upload folder to ~/vtigerlive/user_privileges/
cd ~/vtigerlive/user_privileges/
sudo cp * /var/www/html/vtigercrm/user_privileges/

cd /var/www/html/
sudo chown -R www-data:www-data /var/www/html/vtigercrm
sudo chmod -R 755 /var/www/html/vtigercrm
```

### If ERROR: ERROR EXTRACTING MIGRATION ZIP FILE

<https://discussions.vtiger.com/discussion/185176/error-extracting-migration-zip-file-two-test-instances>

- Upload file: vtigercrm-70x-710-patch.zip

```sh
cd
unzip vtigercrm-70x-710-patch.zip -d vtigercrm-70x-710-patch
cd vtigercrm-70x-710-patch/
unzip vtiger7.zip -d vtiger7

sudo apt install zip -y
zip -r vtiger7.zip vtiger7
sudo cp vtiger7.zip /var/www/html/vtigercrm
```

## Further Improvements to Default Installation

### Install PDF Maker Free

Install PDF Maker Free (basic) from: <https://www.its4you.sk/en/vtiger-shop/extensions-for-vtiger-crm-7-x>
Email: jorge.valente@jv-tech.com

Download for: vtiger CRM Open Source 7.1

### Update TCPDF to latest version 5 (full replacement)

Update: libraries/tcpdf

- Delete tcpdf from 7.1
- Copy everything from lates tcpdf 5.x to vtiger 7.1
- Update File permissions

```sh
sudo rm -R /var/www/html/vtigercrm/libraries/tcpdf
sudo cp -r tcpdf /var/www/html/vtigercrm/libraries/
cd /var/www/html/
sudo chown -R www-data:www-data /var/www/html/vtigercrm
sudo chmod -R 755 /var/www/html/vtigercrm
```

### Extensions to add (Paid)

Consider buying these extensions as they are very useful from vendor <https://it-solutions4you.com/>

- Cashflow/Payment extension 49 Euros
- Delivery Notes + Outgoing Products 59 Euros
- OR buy Extended package for 259 Euros

