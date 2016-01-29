# Provisioning Ubuntu LAMP servers for production

A guide to provisioning a Ubuntu 14.04 Digital Ocean LAMP droplet with Apache, PHP, MySQL, PostFix, Git, Composer, WP-CLI, Symfony Installer and Webmin.

## Digital Ocean Configuration

1. [Add your public SSH keys](https://www.digitalocean.com/community/articles/how-to-use-ssh-keys-with-digitalocean-droplets) to your account Digital Ocean account
2. Create a "LAMP on Ubuntu 14.04" application droplet
3. [Add DNS entries](https://www.digitalocean.com/community/articles/how-to-set-up-a-host-name-with-digitalocean) if using Digital Ocean as name servers


## User Provisioning

#### Reset root password

```
ssh root@mydomain.com
passwd
```

#### Create new user with sudo privileges

```
adduser myusername
gpasswd -a myusername sudo
exit
```

#### Copy authorized keys from root user to new user to enable login by public key

```
ssh myusername@mydomain.com
cd ~
mkdir .ssh
sudo cp /root/.ssh/authorized_keys ~/.ssh/
sudo chmod 700 .ssh
sudo chmod 600 .ssh/authorized_keys
sudo chown myusername: .ssh/authorized_keys
```

#### Create an SSH key pair for your user

```
ssh-keygen -t rsa -C "myemail@mydomain.com"
```

#### Install [Keychain](http://www.funtoo.org/Keychain) to automate ssh-agent

```
sudo apt-get install keychain
```

Edit bash config:

```
nano ~/.bashrc
```

Add lines:

```
alias git='eval $(/usr/bin/keychain --eval --agents ssh -Q --quiet ~/.ssh/*_rsa) && git'
```

## Server Provisioning

#### Upgrade all packages

```
sudo apt-get update
sudo apt-get upgrade
```

#### Secure MySQL

```
mysql_secure_installation
```

#### Create a new MySQL user
```
mysql -u root -p
CREATE USER 'newuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON * . * TO 'newuser'@'localhost';
FLUSH PRIVILEGES;
```

#### Configure Apache


Enable Mod Rewrite and Virtual Hosts:

```
sudo a2enmod rewrite
sudo a2enmod vhost_alias
```

Fix fully qualified domain warning when starting Apache:

```
sudo nano /etc/apache2/apache2.conf
```

Add the line:

```
ServerName localhost
```

#### Configure PHP

Edit PHP Apache module config: 

```
sudo nano /etc/php5/apache2/php.ini
```

Change lines to:

```
date.timezone = America/Chicago   
upload_max_filesize = 8M   
post_max_size = 10M   
```

#### Install Git

```
sudo apt-get install git
git config --global user.name "My Name"
git config --global user.email "myemail@mydomain.com"
```

> Add your user's public SSH key (~/.ssh/id_rsa.pub) to your Git repo's deployment keys to enable passwordless deployment via SSH.

#### Install PostFix

```
sudo apt-get install postfix
```

> Choose the "Internet Site and populate domain using FQDN" option when prompted.

Edit PostFix config:

```
sudo nano /etc/postfix/main.cf
```

Change lines to:

```
myhostname = yourdomain.com
```
Restart PostFix:

```
sudo service postfix restart
```

## Additional Server Provisioning
> These utilities make managing the server easier and are in some cases required for the deployment of WordPress and Symfony applications.

#### Install Webmin (Optional)

Edit apt sources:

```
sudo nano /etc/apt/sources.list
```

Add Lines:

```
deb http://download.webmin.com/download/repository sarge contrib
deb http://webmin.mirror.somersettechsolutions.co.uk/repository sarge contrib
```

Add the Webmin GPG key to apt to trust the source repository

```
wget -q http://www.webmin.com/jcameron-key.asc -O- | sudo apt-key add -
```

Update apt and install Webmin

```
sudo apt-get update
sudo apt-get install webmin
```

> I highly recommend installing [BWTheme](http://theme.winfuture.it/) to spruce things up a bit.

#### Install virtual host management shell script (Optional)
[This shell script](https://github.com/nlenkowski/virtualhost) enables easy management of virtual hosts or [configure your virtual hosts manually](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts).

```
git clone https://github.com/nlenkowski/virtualhost.git && cd virtualhost
chmod +x virtualhost.sh
sudo mv virtualhost.sh /usr/local/bin/vhost
cd ../
sudo rm -R virtualhost
```

#### Install WP-CLI (Optional)
Install [WP-CLI](http://wp-cli.org/) for automating WordPress installations.

```
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
chmod +x wp-cli.phar
sudo mv wp-cli.phar /usr/local/bin/wp
```

#### Install PHP intl and curl modules (Symfony Requirement)

```
sudo apt-get install php5-intl
sudo apt-get install php5-curl
sudo service apache2 restart
```

#### Enable ACLs in the file system (Symfony Requirement)
```
sudo apt-get install acl
```
Follow the remaining instructions [here](https://help.ubuntu.com/community/FilePermissionsACLs).

#### Install Composer (Symfony Requirement)

```
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
```

#### Install the Symfony Installer (Symfony Requirement)

```
sudo curl -LsS http://symfony.com/installer -o /usr/local/bin/symfony
sudo chmod a+x /usr/local/bin/symfony
```

## Server Hardening

For production servers refer to [Hardening a LAMP server](https://docs.google.com/a/littlebiglab.com/document/d/1DSjfbrn1_7MzB5ol7ie_nlOvWA6BENbuHpSUOEITDB0/edit#) documentation.

> **Important:** Make sure to keep a terminal session open when modifying your SSH config and Firewall rules to avoid accidentally locking yourself out.


#### Disable root SSH access

Edit SSH config: 

```
sudo nano /etc/ssh/sshd_config
```

Change line to:

```
PermitRootLogin no
```


Restart SSH service:

```
sudo service ssh restart
```

#### Restrict how much information Apache and PHP expose

Edit Apache security config: 

```
sudo nano /etc/apache2/conf-enabled/security.conf
```

Change lines to:

```
ServerTokens Prod
ServerSignature Off
```

Edit PHP config: 

```
sudo nano /etc/php5/apache2/php.ini
```

Change lines to:

```
expose_php = off 
```

#### Make sure directory listing is disabled in the Apache default host

Edit Apache config:

```
sudo nano /etc/apache2/apache2.conf
``` 

Change lines to:   

```
<Directory /var/www/>
    Options FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```

Restart Apache:

```
sudo apachectl restart
```

#### Configure iptables firewall to allow only SSH and HTTP/HTTPS traffic

```
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport ssh -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
sudo iptables -A INPUT -j DROP
sudo iptables -I INPUT 1 -i lo -j ACCEPT
```

#### Persist iptables rules on reboot

```
sudo apt-get install -y iptables-persistent
sudo service iptables-persistent start
```

## Directory Permissions

> Here's the [canonical answer regarding server permissions](http://serverfault.com/questions/357108/what-permissions-should-my-website-files-folders-have-on-a-linux-webserver) on Serverfault. These permission guidelines are gleaned from that answer.

**Default document root should be owned by root**

```
sudo chown root:root /var/www
```

**Virtual host document roots should be owned by your user and the apache group**

```
sudo chown bblndev:www-data /var/www/myproject
```

**Virtual host document roots should be group writable**

```
sudo chmod 775 /var/www/myproject
```

**Force all new directories and files to inherit the vhost document root group**

```
sudo chmod g+s /var/www/myproject
```

**Force all new files created in the vhost document root to be group writable**

```
sudo umask 0002 /var/www/myproject
```

**Add your user to the apache and adm groups**

```
sudo usermod -aG www-data youruser
sudo usermod -aG adm youruser
```

**To give Apache write access to a directory it should owned by the apache user**

```
sudo chown -R www-data /var/www/myproject/uploads
```
