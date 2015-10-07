# Installing Symfony

Installing and managing Symfony applications on UNIX-like environments (OS X, Linux, FreeBSD, etc).

## Install Symfony

#### Install Composer

[Composer](https://getcomposer.org/) is a PHP package manager and is required for Symfony development.

```
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
```

#### Install Symfony installer

```
sudo curl -LsS http://symfony.com/installer -o /usr/local/bin/symfony
sudo chmod a+x /usr/local/bin/symfony
```

#### Change to project directory

```
cd /var/www/myproject
```

#### Create a new Symfony project

```
symfony new project
```

#### Cleanup after install

```
mv project/* ./ && mv project/.[^.]* ./ && rmdir project
```

#### Set application permissions

Different environments require different methods for setting up permissions. See [Setting up Permissions](http://symfony.com/doc/current/book/installation.html#book-installation-permissions).

#### Install application

```
composer install
```

#### Update database schema

```
php app/console doctrine:schema:update --force
```

#### Import data fixtures

```
php app/console doctrine:fixtures:load --append
```

#### Clear caches

```
php app/console assetic:dump --env=dev --no-debug
php app/console cache:clear --env=dev
php app/console assetic:dump --env=prod --no-debug
php app/console cache:clear --env=prod --no-debug
```

#### Make sure it works
If all is well you should now be able to access the site at [http://myproject.com/web/](http://example.com/web/). If you want to remove the /web requirement from the URL you'll need to change your site's document root to /var/www/myproject/web/.