# Creating a Rails application
This is [a solid guide](http://railsapps.github.io/installrubyonrails-mac.html) for getting Ruby, RVM and Rails working together happily and creating your first Rails app.
 
#### Create a new RVM Gemset for your app and choose which version of Ruby to use   

```
mkdir /var/www/myrailsapp
cd /var/www/myrailsapp
rvm use ruby-2.1.4@myapp --ruby-version --create
```

#### Install Rails and create a new Rails app
```
gem install rails
rails new .
```