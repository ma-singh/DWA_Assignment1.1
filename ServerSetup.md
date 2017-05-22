## Preparing a Server
#### Prerequisites

To begin with, you should have a non-root user account on the server with ```sudo``` privileges

***
Before installing Nginx or any other software to the server, make sure you update Ubuntu's packages.

```
$ sudo apt-get update

$ sudo apt-get install nginx
```
Now we need to allow connections to Nginx:

```
sudo ufw allow 'Nginx HTTP'
```
### Install MariaDB

Now we install MariaDB:

```
sudo apt install mariadb-server mariadb-client
```
To secure the installation, we'll run a script: 

```
sudo mysql_secure_installation
```
We haven't set a root password yet, so press *enter*, and then press *y* to set a root password. Answer **y** for **yes** to the remaining questions:

> Remove anonymous users? **yes**

> Disallow root login remotely? **yes**

> Remove test database and access to it? **yes**

> Reload privilege tables now? **yes**

### Install PHP

The next step is to install **PHP7** and some of its extensions. Begin by installing PHP with: 

```
sudo apt-get install php-fpm php-mysql
```
Change the `php-fpm` configuration file by running

```
sudo nano /etc/php/7.0/fpm/php.ini
```

Within the file, find the `cgi.fix_pathinfo` parameter, uncomment it and set to `cgi.fix_pathinfo=0` from `1`. Save and close the file.

Restart PHP by running: 

```
sudo systemctl restart php7.0-fpm
```

To configure Nginx to use the PHP Processor, we'll open the Nginx server block configuration with:

```
sudo nano /etc/nginx/sites-available/default
```

Update the file to match in the following locations:

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;
    index **INDEX.PHP** index.html index.htm index.nginx-debian.html;

    server_name **IP_ADDRESS**;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.0-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```
Save and close the file when complete. Check for any errors by running: 

```
sudo nginx -t
```

If there are no errors, reload Nginx with: 

```
sudo systemctl reload nginx
```

#### Install Additional PHP Extensions

Install additional PHP extensions by running: 

```
sudo apt-get install php-curl php-gd php-mbstring php-mcrypt php-xml php-xmlrpc
```

Once complete, restart PHP with: 

```
sudo systemctl restart php7.0-fpm
```

#### Testing 

Check that everything is working properly by creating a test file by running:

```
sudo nano /var/www/html/info.php
```
Inside this file, simply write out the following: 

```
<?php
phpinfo();
?>
```
Then save and close the file. Visit that page in your browser at: 

```
ip_address/info.php
```
And you should see PHP generated information about your server. For security's sake, remove this file after testing by running: 

```
sudo rm /var/www/html/info.php
```

### Setting Up MariaDB
Log in to MariaDB with 

```
mysql -u root -p
```
Enter the password you created when setting up the account.

To create a database in MariaDB, use: 
 
``` 
CREATE DATABASE dwa_w1d1 DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
```

Next, create a User with a password and give it access to the database

``` 
GRANT ALL ON dwa_w1d1.* TO 'dwa_user'@'localhost' IDENTIFIED BY 'password'; 
```

Notify MariaDB that you've made changes by running: 

``` 
FLUSH PRIVILEGES; 
```

#### Using MariaDB

You can start MariaDB with the command:

```
sudo systemctl start mariadb
```

To check whether MariaDB is running, use the following command:

```
sudo systemctl status mariadb
```

You can exit MariaDB with either `EXIT;` or `sudo systemctl stop mariadb`.

### Adjust Nginx to Handle WordPress

Open the Nginx's default server block files with: 

``` 
sudo nano /etc/nginx/sites-available/default 
```

Add the following new locations in the server block beneath the existing ones: 

``` 
	location = /favicon.ico { log_not_found off; access_log off; }
	location = /robots.txt { log_not_found off; access_log off; allow all; }
	location ~* \.(css|gif|ico|jpeg|jpg|js|png)$ {
        expires max;
        log_not_found off;
    }
```
In the existing try_files block, edit it to the following: 

```
location / {
    #try_files $uri $uri/ =404;
    try_files $uri $uri/ /index.php$is_args$args;
} 
```

When you're finished, save and close the file; check for errors by running: 

```
sudo nginx -t
```

If there were no errors, reload Nginx with: 

```
sudo systemctl reload nginx
```

## Downloading WordPress

Within your server, create a new empty directory and change into it so you can download the latest release of WordPress.

```
$ cd /tmp
$ curl -O https://wordpress.org/latest.tar.gz
```

Extract the contents from the file you just downloaded

```
tar xzvf latest.tar.gz
```

Make a copy of the sample config file and direct it as the file that WordPress actually uses

```
cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
```
Create an upgrade directory so that WordPress will be able to update without running into permission issues later on.

```
mkdir /tmp/wordpress/wp-content/upgrade
```
Copy the contents into the document root. The use of `-a` maintains permissions, and the dot at the end of the source directory copies everything in the directory, including hidden files

```
sudo cp -a /tmp/wordpress/. /var/www/html
```
### Configure WordPress Directory

Assign ownership of all files in the document root to the `sudo` username that we've created

```
sudo chown -R nashsingh:www-data /var/www/html
```

Make sure the server has group ownership over files created in the directory

```
sudo find /var/www/html -type d -exec chmod g+s {} \;
```

Give group write access to `wp-content` to allow the web interface to make theme and plugin changes

```
sudo chmod g+w /var/www/html/wp-content
```

Follow up by giving the server write access to what's in those two directories

```
sudo chmod -R g+w /var/www/html/wp-content/themes
sudo chmod -R g+w /var/www/html/wp-content/plugins
```
#### Setting Up the WordPress Configuration File

First we need to provide security for our installation. Grab secret keys generated by WordPress after running the following command and save them somewhere

```
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```

Copy the results generated, then open up the WordPress configuration file: 

```
nano /var/www/html/wp-config.php
```

Find and delete the placeholder data for your authentication keys and salts with the data you previously copied that was generated by WordPress

```
define('AUTH_KEY',         'YOURDATAGOESHEREYOURDATAGOESHEREYOURDATAGOESHEREYOURDATAGOESHERE');
define('SECURE_AUTH_KEY',  'YOURDATAGOESHEREYOURDATAGOESHEREYOURDATAGOESHEREYOURDATAGOESHERE');
define('LOGGED_IN_KEY',    'YOURDATAGOESHEREYOURDATAGOESHEREYOURDATAGOESHEREYOURDATAGOESHERE');
define('NONCE_KEY',        'YOURDATAGOESHEREYOURDATAGOESHEREYOURDATAGOESHEREYOURDATAGOESHERE');
define('AUTH_SALT',        'YOURDATAGOESHEREYOURDATAGOESHEREYOURDATAGOESHEREYOURDATAGOESHERE');
define('SECURE_AUTH_SALT', 'YOURDATAGOESHEREYOURDATAGOESHEREYOURDATAGOESHEREYOURDATAGOESHERE');
define('LOGGED_IN_SALT',   'YOURDATAGOESHEREYOURDATAGOESHEREYOURDATAGOESHEREYOURDATAGOESHERE');
define('NONCE_SALT',       'YOURDATAGOESHEREYOURDATAGOESHEREYOURDATAGOESHEREYOURDATAGOESHERE');
```

Further up in the file, change the lines in your database connection settings to match what you previously created in MariaDB.

```
/** The name of the database for WordPress */
define('DB_NAME', 'dwa_w1d1');

/** MySQL database username */
define('DB_USER', 'dwa_user');

/** MySQL database password */
define('DB_PASSWORD', 'password');
```
Underneath these, also add in the line
 
```
define('FS_METHOD', 'direct');
```
Save and close the file when complete.

## Complete the Installation
Finish the installation in your browser by visiting the IP address

```
http://104.236.87.252/
```
Select the language you would like to use (we're using English), and then finish the rest of the setup guide by creating a domain name, a username and password, and an email address.
