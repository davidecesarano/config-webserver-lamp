# Webserver LAMP - Ubuntu 14.04

## Software

### Apache

Install Apache:
```
$ sudo apt-get update
$ sudo apt-get -y install apache2
```

Active rewrite, headrs and ssl modules:
```
$ sudo a2enmod rewrite
$ sudo a2enmod headers
$ sudo a2enmod ssl
```

Restart Apache:
```
$ sudo service apache2 restart
```

### PHP

Install PHP and modules:
```
$ sudo apt-get update
$ sudo apt-get -y install php5-mysql php5 libapache2-mod-php5 php5-mcrypt php5-cli php5-gd php5-curl php5-xcache
```

Open the *dir.conf* file:
```
$ sudo nano /etc/apache2/mods-enabled/dir.conf
```

Move the PHP index file above to the first position after the DirectoryIndex specification:
```
<IfModule mod_dir.c>
    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

Create root directory and index.php file:
```
$ sudo mkdir /var/www/html/root
$ sudo touch /var/www/html/root/index.php
$ echo "<?php echo '<h1>It\'s Works!</h1>'; ?>" > /var/www/html/root/index.php
```

Open the *000-default.conf* file:
```
$ sudo nano /etc/apache2/sites-available/000-default.conf
```

Clear all and add:
```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/root
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    <Directory /var/www/html/root>
        Options -Indexes -Includes
        Order allow,deny
        Allow from all
    </Directory>
</VirtualHost>  
```

Restart Apache:
```
$ sudo service apache2 restart
```

### MySQL

Install MySQL and follow the wizard:
```
$ sudo apt-get update
$ sudo apt-get -y install mysql-server
```

### PhpMyAdmin

Install PhpMyAdmin, follow the wizard and enable php5-mcrypt:
```
$ sudo apt-get install phpmyadmin
$ sudo php5enmod mcrypt
```

Restart Apache:
```
$ sudo service apache2 restart
```

### ACL

```
$ sudo apt-get install acl
```

### Mailutils

```
$ sudo apt-get install mailutils
```

### SFTP

Add the sftp group:
```
$ sudo addgroup sftp
```

Open the *sshd_config* file:
```
$ sudo nano /etc/ssh/sshd_config
```

Find *Subsystem*, comment it and add new: 
```
#Subsystem sftp /usr/lib/openssh/sftp-server
Subsystem sftp internal-sftp
```

After ... add:
```
Match Group sftp
ChrootDirectory %h/domains/
X11Forwarding no
AllowTcpForwarding no
ForceCommand internal-sftp
```

Restart SSH:
```
$ sudo service ssh restart
```

### Git and Composer

Install Git and Composer:
```
$ sudo apt-get -y install git
$ curl -s https://getcomposer.org/installer | php
$ sudo mv composer.phar /usr/local/bin/composer
```

## Security

### Apache

ServerSignature Off
ServerTokens Prod

### PHP

```
$ sudo nano /etc/apach2/conf-available/security.conf 
```

```
    ServerTokens Prod
    ServerSignature Off
    Header always append X-Frame-Options SAMEORIGIN
    Header set X-XSS-Protection: "1; mode=block"
    Header edit Set-Cookie ^(.*)$ $1;HttpOnly;Secure
    Timeout 60
```

### PhpMyAdmin

Open the config.inc.php file:
```
$ sudo nano /etc/phpmyadmin/config.inc.php
```

After *$cfg['Servers'][$i]['recent']* add:
```
$cfg['Servers'][$i]['hide_db'] = '^information_schema|mysql|performance_schema|phpmyadmin$';
```

After *$cfg['SaveDir']* add:
```
$cfg['ForceSSL'] = true;
```

Open the *apache.conf* file:
```
$ sudo nano /etc/phpmyadmin/apache.conf
```

Find */phpmyadmin* and replace with:
```
Alias /database/domains /usr/share/phpmyadmin
```

Restart Apache:
```
$ sudo service apache2 restart
```

## Users and Domains

### Create User

Create user, set home folder and add user to www-data and sftp groups:
```
$ useradd -m -d /var/www/html/USERNAME -s /bin/false -g www-data -G sftp USERNAME
$ passwd USERNAME
```

### Create Folders

Set root user and root group for home user folder:
```
$ sudo chown root:root /var/www/html/USERNAME
```

Create domains folder in home user folder:
```
$ mkdir /var/www/html/USERNAME/domains
```

Create domain (EXAMPLE.COM) folder in domains folder:
```
$ mkdir /var/www/html/USERNAME/domains/EXAMPLE.COM
```

Create httpdocs, logs and backups folders in EXAMPLE.COM folder:
```
$ mkdir -p /var/www/html/USERNAME/domains/EXAMPLE.COM/{httpdocs,logs,backups}
```

Set permissions:
```
$ setfacl -R -m u:USERNAME:rwx /var/www/html/USERNAME/domains/EXAMPLE.COM/httpdocs
$ setfacl -Rd -m u:USERNAME:rwx /var/www/html/USERNAME/domains/EXAMPLE.COM/httpdocs
$ setfacl -R -m g:www-data:rwx /var/www/html/USERNAME/domains/EXAMPLE.COM/httpdocs
$ setfacl -Rd -m g:www-data:rwx /var/www/html/USERNAME/domains/EXAMPLE.COM/httpdocs
```

Create index.php file in httpdocs directory and set permissions:
```
$ sudo touch /var/www/html/USERNAME/domains/EXAMPLE.COM/httpdocs/index.php
$ echo "<?php echo '<h1>It\'s Works!</h1>'; ?>" > /var/www/html/USERNAME/domains/EXAMPLE.COM/httpdocs/index.php
$ sudo chown USERNAME:www-data /var/www/html/USERNAME/domains/EXAMPLE.COM/httpdocs/index.php
```

### Create Virtual Host

### Create Backup
