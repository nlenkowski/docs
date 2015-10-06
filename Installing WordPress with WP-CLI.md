# Installing WordPress with WP-CLI

Installing WP-CLI and WordPress on UNIX-like environments (OS X, Linux, FreeBSD).


## Install WP-CLI

[WP-CLI](http://wp-cli.org/) provides command line management for nearly [any WordPress task](http://wp-cli.org/commands/). Install WordPress core, add plugins, themes, users, posts, etc, all without leaving the terminal.

#### Install WP-CLI

```
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
```

#### Make it executable and move it to somewhere in your path

```
chmod +x wp-cli.phar
sudo mv wp-cli.phar /usr/local/bin/wp
```

## Install WordPress using WP-CLI

#### Change to project directory

```
cd /var/www/myproject
```

#### Install WordPress
> Add appropriate values for each parameter in the config and install commands

```
wp core download &&
wp core config --dbname=mydatabase --dbuser=bblndev --dbpass=mydbpass &&
wp core install --url=http://myproject.com --title=WordPress --admin_user=admin --admin_password=password --admin_email=admin@bblndev.com
```

#### Install plugins

```
wp plugin install hello-dolly
```

#### Install themes

```
wp theme install https://github.com/nlenkowski/blujay/archive/master.zip
```

#### Complete installation
That's it! Hooray for WP-CLI!

## Install WordPress using the web installer

For those that like to kick it old school.

#### Change to project directory

```
cd /var/www/myproject
```

#### Install WordPress

```
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
```

#### Cleanup install files

```
mv wordpress/* ./ && rmdir wordpress && rm latest.tar.gz
```

#### Complete installation
You'll need to complete the installation via the web installer now.