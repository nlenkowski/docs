# Provisioning OS X for web development

> Updated for OS X 10.11 El Capitan

### A guide to provisioning:

- Prezto
- Homebrew
- Git
- Apache
- PHP
- MySQL
- PostgreSQL
- Node and npm
- Ruby and RVM
- Gulp, Bower, WP-CLI, Composer, etc.

## Install Prezto
We've got a lot of work to do in the terminal, so lets make it awesome first.  

[Install Prezto](https://github.com/sorin-ionescu/prezto) and [make it sweet](http://mikebuss.com/2014/02/02/a-beautiful-productive-terminal-experience/).

## Install Homebrew
We'll be using [Homebrew](http://brew.sh/) to do our heavy lifting.

#### Install the Xcode Command Line Tools

```
xcode-select --install
```

#### Install Homebrew

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

#### Make sure we're ready to brew

```
brew doctor
```

## Install Git

Git comes preinstalled with OS X, however its outdated and doesn't support auto-completion. We'll install a standalone Git with Homebrew.

#### Install Git with Homebrew

```
brew install git
```

#### Configure Git

```
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

#### Enable Git over SSH

> The primary advantage of working with Git over SSH (vs HTTPS) is that you'll never have to enter your remote repository credentials while working, its all handled by SSH and your public key.

#### Generate an SSH key for your user

```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

#### Copy your public key

```
pbcopy < ~/.ssh/id_rsa.pub
```

#### Add your public key to your Git hosting service accounts

Now fire up your web browser and add your public key to your Git hosting accounts (GitHub, BitBucket, etc).

#### Make sure it works

You should be able to clone remote repositories via SSH without being prompted for a password. You can also try connecting to Github via SSH. If the username in the message is yours, you've successfully set up your SSH key.

```
ssh -T git@github.com
```

## Configure Apache

I've never had the need to upgrade Apache between OS X releases and installing it with Homebrew is somewhat of a pain, so we'll use the Apache installation that ships with OS X instead.

> Note that upgrading OS X will overwrite your Apache config, so make sure to create a backup before upgrading.

#### Edit Apache config

```
sudo nano /etc/apache2/httpd.conf
```

#### Change the Apache user and group

```
User your_user
Group staff
```

#### Change the default directory permissions

From

```
<Directory />
    AllowOverride none
    Require all denied
</Directory>
```

To

```
<Directory />
    Options Indexes MultiViews FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```

> Running Apache as your user and loosening default directory permissions is not recommended in production environments, however this being a local development environment we'll make an exception so file permissions never give us any trouble.

#### Enable rewrite, virtual hosts and php modules

Uncomment the following lines:

```
LoadModule rewrite_module libexec/apache2/mod_rewrite.so
LoadModule php5_module libexec/apache2/libphp5.so
Include /private/etc/apache2/extra/httpd-vhosts.conf
```

#### Edit vhosts config

```
sudo nano /etc/apache2/extra/httpd-vhosts.conf
```

#### Add some vhosts

We'll assume you're webroot is /users/your_user/Sites.

```
<VirtualHost *:80>
    DocumentRoot "/users/your_user/Sites"
</VirtualHost>

<VirtualHost *:80>
    ServerName mysite.dev
    DocumentRoot "/users/your_user/Sites/mysite"
</VirtualHost>
```

#### Make sure our config checks out and restart apache

```
sudo apachectl configtest 
sudo apachectl restart
```

## Enable SSL for Apache (Optional)
If you want to work with secure sites (HTTPS) locally you'll need to configure SSL for Apache.

#### Generate a self-signed certificate:

```
sudo mkdir /etc/apache2/ssl
sudo ssh-keygen -f host.key 
sudo openssl req -new -key host.key -out request.csr
sudo openssl x509 -req -days 365 -in request.csr -signkey host.key -out server.crt 
sudo openssl rsa -in host.key -out host.nopass.key
```

#### Edit Apache config

```
sudo nano /etc/apache2/httpd.conf
```

#### Uncomment the following lines

```
LoadModule ssl_module libexec/apache2/mod_ssl.so
LoadModule socache_shmcb_module libexec/apache2/mod_socache_shmcb.so
Include /private/etc/apache2/extra/httpd-ssl.conf
```

#### Edit Apache SSL config

```
sudo nano /etc/apache2/extra/httpd-ssl.conf
```

#### Update the following lines as necessary

```
SSLEngine on 
SSLCertificateFile "/private/etc/apache2/ssl/server.crt"
SSLCertificateKeyFile "/private/etc/apache2/ssl/host.nopass.key"
```

#### Edit vhosts config

```
sudo nano /etc/apache2/httpd-vhosts.conf
```

#### Add an SSL enabled vhost

```
<VirtualHost *:443>
    SSLEngine on
    SSLCertificateFile "/private/etc/apache2/ssl/server.crt"
    SSLCertificateKeyFile "/private/etc/apache2/ssl/host.nopass.key"
    ServerName mysite.dev
    DocumentRoot "/var/www/mysite"
</VirtualHost>
```

#### Make sure our config checks out and restart apache
```
sudo apachectl configtest 
sudo apachectl restart 
```

## Install PHP

OS X ships with PHP, however using Homebrew to install a standalone PHP makes installing extensions and multiple PHP versions a piece of cake. Here's a nice guide to [switching PHP versions](http://getgrav.org/blog/mac-os-x-apache-setup-multiple-php-versions) on the fly.

#### Install PHP with Homebrew

```
brew tap homebrew/dupes
brew tap homebrew/versions
brew tap homebrew/homebrew-php
brew install php56
```

#### Edit Apache config

```
sudo nano /etc/apache2/http.conf
```

#### Update the following line to so Apache will load our new standalone PHP

```
LoadModule php5_module /usr/local/opt/php55/libexec/apache2/libphp5.so
```

#### Make sure we're using our standalone PHP on the command line as well

```
export PATH="$(brew --prefix homebrew/php/php56)/bin:$PATH"
```

#### Edit php.ini

```
sudo nano /usr/local/etc/php/5.6/php.ini
```

#### Update your config as appropriate

```
date.timezone = "Europe/London"
upload_max_filesize = 100M
post_max_size = 100M
```

#### Restart Apache

```
sudo apachectl restart
```

## Install MySQL

#### Install MySQL with Homebrew

```
brew install mysql
```

#### Start MySQL now and on boot

```
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
ln -sfv /usr/local/opt/mysql/*.plist ~/Library/LaunchAgents
```

#### Secure MySQL

```
mysql_secure_installation
```

## Install PostgreSQL

#### Install PostgreSQL with Homebrew

```
brew install postgresql
```

#### Start PostgreSQL now and on boot

```
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist
ln -sfv /usr/local/opt/postgresql/*.plist ~/Library/LaunchAgents
```

## Install Ruby and RVM
Ruby ships with OS X, however its not possible to switch Ruby versions easily and not recommended to mess around with your system Ruby in general. We'll use RVM to manage and install Ruby instead. 

> This is [a good guide](http://foffer.dk/install-ruby-on-os-x-10-10-yosemite-using-rvm/) for installing Ruby and RVM on OS X.

#### Install RVM

```
\curl -sSL https://get.rvm.io | bash -s stable
```

#### Enable RVM in the current terminal session

```
source /Users/nathan/.rvm/scripts/rvm
```

#### Install Ruby

```
rvm install ruby
```

#### Set the default Ruby

```
rvm --default use ruby-2.1.4
```

## Install Node

#### Install Node and npm

```
brew install node
```

## Install some useful command line tools

#### Install Gulp, Grunt, Bower and Yeoman

```
npm install -g gulp
npm install -g grunt-cli
npm install -g bower
npm install -g yo
```

#### Install WP-CLI

```
brew install wp-cli
```

#### Install Composer

```
brew install composer
```
