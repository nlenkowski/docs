# Provisioning macOS for web development

> Updated for macOS 10.12 Sierra

### A guide to provisioning:

- Prezto
- Homebrew
- Git
- Apache
- PHP
- MySQL
- PostgreSQL
- Node
- rbenv and Ruby
- Some useful CLI tools

## Prezto
We've got a lot of work to do in the terminal, so lets make it awesome first.

[Install Prezto](https://github.com/sorin-ionescu/prezto) and [make it sweet](http://mikebuss.com/2014/02/02/a-beautiful-productive-terminal-experience/).

## Homebrew
We'll be using [Homebrew](http://brew.sh/) to do our heavy lifting.

#### Install Command Line Tools

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

#### Tap additional formulae repositories and update homebrew

```
brew tap homebrew/dupes
brew tap homebrew/versions
brew tap homebrew/homebrew-php
brew update
```
> These repositories facilitate installing duplicate versions of system utilities and PHP.

## Git

Git comes preinstalled on macOS, however its outdated and doesn't support auto-completion. We'll install homebrew Git in its place.

#### Install Git

```
brew install git
```

#### Configure Git

```
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

#### Generate an SSH key pair

```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

#### Copy your public key to the clipboard

```
pbcopy < ~/.ssh/id_rsa.pub
```

#### Add your public key to your Github account

[Add your public key](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/) to your Github account and make sure you can connect via SSH.

```
ssh -T git@github.com
```

## Apache

As of macOS Sierra its no longer possible to build PHP against the system Apache. Additionally, the system Apache configuration is overwritten when upgrading macOS. To avoid these issues we'll use homebrew Apache instead.

#### Disable system Apache
```
sudo apachectl stop
sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist 2>/dev/null
```

#### Install homebrew Apache

```
brew install httpd24 --with-privileged-ports
```

#### Configure homebrew Apache

```
sudo nano /usr/local/etc/apache2/2.4/httpd.conf
```

#### Change the Apache process user and group

Change:

```
FROM:

User daemon
Group daemon

TO:
 
User your_user
Group staff
```

> Running Apache as your user is not recommended in production environments for security reasons, however we'll make an exception so file permissions don't give us any trouble on our development box.

#### Change the default directory permissions

Change:

```
FROM:

<Directory />
    AllowOverride none
    Require all denied
</Directory>

TO:

<Directory />
    Options FollowSymLinks
    AllowOverride all
    Require all granted
</Directory>
```

#### Enable deflate, expires and rewrite modules

Uncomment:

```
LoadModule deflate_module libexec/mod_deflate.so
LoadModule expires_module libexec/mod_expires.so
LoadModule rewrite_module libexec/mod_rewrite.so
```

Add:

```
<ifmodule mod_deflate.c>
	AddOutputFilterByType DEFLATE text/html text/plain text/xml text/css text/javascript application/javascript
</ifmodule>
```

#### Enable virtual hosts

Uncomment:

```
Include /usr/local/etc/apache2/2.4/extra/httpd-vhosts.conf
```

#### Configure virtual hosts

```
sudo nano /usr/local/etc/apache2/2.4/extra/httpd-vhosts.conf
```

Add:

```
<VirtualHost *:80>
    DocumentRoot "/Users/your_user/Sites"
</VirtualHost>
```

#### Restart apache

```
sudo brew services restart httpd24
```

You should now be able to view your webroot at [http://localhost](http://localhost)

> By design non-root applications on macOS can not listen on any port below 1024. The --with-privileged-ports option allows Apache to listen on port 80, however its LaunchDaemon needs to be started as root, hence sudo is required to manage the Apache service. 
> 
> Here is [an alternate method](https://goo.gl/otZv5o) of configuring Apache to listen on port 80 by forwarding port 8080 => 80.

## PHP

MacOS ships with PHP, however using homebrew PHP makes installing extensions and switching PHP versions a piece of cake.

#### Install PHP

```
brew install php70 --with-apache
```

#### Install optional PHP modules, eg:

```
brew install php70-apcu
brew install php70-opcache
brew install php70-imagick
```

#### Enable PHP 

Edit Apache config:

```
sudo nano /usr/local/etc/apache2/2.4/httpd.conf
```

Add:

```
LoadModule php7_module /usr/local/opt/php70/libexec/apache2/libphp7.so
```

Add:

```
<FilesMatch .php$>
    SetHandler application/x-httpd-php
</FilesMatch>
```

Change:

```
FROM:

DirectoryIndex index.html

TO:

DirectoryIndex index.php index.html
```

#### Enable homebrew PHP on the CLI

```
export PATH="$(brew --prefix homebrew/php/php70)/bin:$PATH"
```

#### Edit php.ini

```
sudo nano /usr/local/etc/php/7.0/php.ini
```

Update as appropriate:

```
date.timezone = "Europe/London"
upload_max_filesize = 64M
post_max_size = 64M
```

#### Restart Apache

```
sudo brew services restart httpd24
```

## Install MySQL

#### Install MySQL

```
brew install mysql
```

#### Start MySQL

```
brew services start mysql
```

#### Secure MySQL

```
mysql_secure_installation
```

## PostgreSQL

#### Install PostgreSQL

```
brew install postgresql
```

#### Start PostgreSQL

```
brew services start postgresql
```

## Install Ruby and rbenv
Ruby ships with macOS, however its not recommended to mess around with your system Ruby. We'll use rbenv to install and manage our Ruby versions instead. 

> This is [a good guide](https://gorails.com/setup/osx/10.12-sierra) for installing rbenv, Ruby and Rails.

#### Install rbenv

```
brew install rbenv
rbenv init
```

#### Add rbenv to your path

If your shell is Bash:

```
echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
```

If your shell is Zsh:

```
echo 'eval "$(rbenv init -)"' >> ~/.zshrc
```

#### Install Ruby

```
rbenv install 2.3.1
rbenv global 2.3.1
ruby -v
```

## Node

#### Install Node and npm

```
brew install node
```

## Some useful CLI tools

#### Install BrowserSync, Grunt and Gulp

```
npm install -g browser-sync
npm install -g grunt-cli
npm install -g gulp
```

#### Install WP-CLI

```
brew install wp-cli
```

#### Install Composer

```
brew install homebrew/php/composer
```


