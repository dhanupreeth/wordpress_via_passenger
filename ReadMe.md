## Wordpress along with Rails app with Apache2 and Passenger

### Intro
This document helps to run Wordpress blog along with Rails app in a same server and make it available / accessible via Rails App. In this example we want to run a Rails app that accsesible via domain name www.dhanesh-example.com and also want to run a Wordpress instance that accessible via /blog. 

1. user access the www.dhanesh-example.com or any paths (except /blog /api), it will refer to the Rails app
2. user access the www.dhanesh-eample.com/blog or any paths inside /blog path scope it will refer to wordpress.

### Steps
- New or Running Rails project path /var/www/html/rails_app
- Install passenger gem
- Install apache2 and passenger module
- Create virtual host for Rails app 
- Download, Extract, and rename the wordpress folder to blog
- Move the blog folder to /var/www/html/blog and configure our wordpress instance
- Create MySQL database for our wordpress instance
- set WP_HOME and WP_SITEURL wordpress config to http://www.dhanesh-example.com/blog
- Add gem 'rails-reverse-proxy' to your Gemfile and run bundle install, Commit the Gemfile and Gemfile.lock
- Create controller to handle reverse proxy from Rails to wordpress
- Create new route that will point access to /blog path to our reverse proxy controller.

### ADDITIONAL DETAILS
Configure Rails, Passenger, Apache2

Install passenger gem and continue with installing apache2 passenger module by run passenger-install-apache2-module. Follow any instructions from the installation guide until all requirements met. 

Note: don't forget the important part from the installation guide that will ask you to load passenger module in your apache2 configuration file /etc/apache2/apache2.conf

Create new virtual host by creating new config file in /etc/apache2/sites-available/dhanesh-example.conf folder. Use this configuration for the virtual host:
```bash
<VirtualHost *:80>
    ServerName dhanesh-example.com
    ServerAlias www.dhanesh-example.com
    ServerAdmin dhanesh@example.com
    DocumentRoot /var/www/html/rails_app/current/public
    RailsEnv production
    ErrorLog /var/log/apache2/app-name/error.log
    CustomLog /var/log/apache2/app-name/production.log combined
    <Directory "/var/www/html/rails_app/current/public">
        Options FollowSymLinks
        Require all granted
    </Directory>
</VirtualHost>
```
### Wordpress Configuration
Update the wp-config.php set WP_HOME and WP_SITEURL to http://www.dhanesh-example.com/blog.

```bash
define('WP_SITEURL', 'http://dhanesh-example.com/blog');
define('WP_HOME',    'http://dhanesh-example.com/blog');
define('FS_METHOD', 'direct');
```
### Configuring Reverse proxy from Rails to Wordpress
Install rails-reverse-proxy gem

Create a controller that will handle reverse proxy from Rails to wordpress. In my example, the controller name is WordpressController with the following codes:

```bash
# app/controllers/wordpress_controller.rb
class WordpressController < ApplicationController
  include ReverseProxy::Controller
  def index
    reverse_proxy "http://dhanesh-example.com:80" do |config|
      config.on_missing do |code, response|
        redirect_to root_url   and return
      end
    end
  end
  end
```
You could change the http://dhanesh-example.com:80 part with any host and any port where your wordpress instance running.

Next, you need to add new route to config/route.rb that will point any request to /blog path in our Rails app to our reverse proxy controller (WordpressController). Here is my config/routes.rb:

```bash
Rails.application.routes.draw do
  match 'blog' => 'wordpress#index', via: [:get, :post, :put, :patch, :delete]
  match 'blog/*path' => 'wordpress#index', via: [:get, :post, :put, :patch, :delete]

  get 'home/index'
  root to: 'home#index'
end
```
When there is request to /blog path, our Rails app will point it to apache server and looking for a folder with name blog, that's why we need to rename our wordpress folder into blog.

### Configure virtual host for wordpress

Edit the previously created config file in dhanesh-example.conf folder. Add the following configurations to the virtual host:
```bash
    Alias /blog /var/www/html/blog
    <Directory /var/www/html/blog>
        Allow From all
        Options +Indexes
        AllowOverride all
    </Directory>

    <Location /blog>
      PassengerEnabled off
    </Location>
```

Restart the Apache server again.

### Test
dhanesh-example.com it will display rails page and try to access dhanesh-example.com/blog it will point to the wordpress blog

### Thank you







