# Laravel configuration with LAMP stack.

This guide will also cover Laravel pretty URLs with `UBUNTU 18.04`. This guide is heavily inspired from [this guide][1]

## Step 1 — Installing Apache and Updating the Firewall

Install Apache using Ubuntu's package manager, `apt`:

```
sudo apt update
sudo apt install apache2
```

### Adjust the Firewall to Allow Web Traffic

```sudo ufw app list```

Output
```
Available applications:
  Apache
  Apache Full
  Apache Secure
  OpenSSH
  ```
  
If you look at the `Apache Full` profile, it should show that it enables traffic to ports `80` and `443`:
```sudo ufw app info "Apache Full"```

```
Output
Profile: Apache Full
Title: Web Server (HTTP,HTTPS)
Description: Apache v2 is the next generation of the omnipresent Apache web
server.

Ports:
  80,443/tcp
```

Allow incoming HTTP and HTTPS traffic for this profile:

```
sudo ufw allow in "Apache Full"
```

Now 
```
http://your_server_ip
```
will serve you `Apache` default page.

## Step 2 — Installing MySQL

```
sudo apt install mysql-server
```

Setup password for `root` user.

```
sudo mysql
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'secure_password';
mysql> FLUSH PRIVILEGES;
mysql> exit;
```
> For secure `MySQL` installation, please check [this guide][1]

## Step 3 — Installing PHP

In original guide, required Modules for `Laravel` are missing. Guide contains something like this.

> ~~sudo apt install php libapache2-mod-php php-mysql~~

instead, we're going to install `PHP 7.2` with all the Modules that `Laravel` requires.

```
sudo apt install php7.2 libapache2-mod-php7.2 php7.2-mbstring php7.2-xmlrpc php7.2-soap php7.2-gd php7.2-xml php7.2-cli php7.2-zip php-mysql php-curl

```

In most cases, you will want to modify the way that Apache serves files when a directory is requested. Currently, if a user requests a directory from the server, `Apache` will first look for a file called `index.html`. We want to tell the web server to prefer `PHP` files over others, so make `Apache` look for an `index.php` file first.

To do this, type this command to open the `dir.conf` file in a text editor with `root` privileges:

```
sudo vi /etc/apache2/mods-enabled/dir.conf
```

```vim
<IfModule mod_dir.c>
    DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
</IfModule>
```

move `index.php` after `DirectoryIndex`

```vim
<IfModule mod_dir.c>
    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

Restart the `Apache` web server.

```
sudo systemctl restart apache2
```

## Step 4: Install Composer to Download Laravel
```
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
```

cd `/var/www/html`
```
sudo composer create-project laravel/laravel MyProject --prefer-dist
```

### Change Permissions

```
sudo chown -R www-data:www-data /var/www/html/MyProject/
sudo chmod -R 755 /var/www/html/MyProject/
```

## Step 5: Configure Apache

You can choose whatever name you want for this file, I am going to stick with `laravel.conf`

```
sudo vi /etc/apache2/sites-available/laravel.conf
```

Add following content to file.
```vim
<VirtualHost *:80>   
  ServerAdmin admin@example.com
     DocumentRoot /var/www/html/MyProject/public
     ServerName example.com

     <Directory /var/www/html/MyProject/public>
        Options +FollowSymlinks
        AllowOverride All
        Require all granted
     </Directory>

     ErrorLog ${APACHE_LOG_DIR}/error.log
     CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Save the file and exit(hit `esc` then type `:wq` and hit `Enter`).

## Step 6: Enable the Laravel and Rewrite Module

```
sudo a2dissite 000-default.conf
sudo a2ensite laravel.conf
sudo a2enmod rewrite
sudo service apache2 restart
```

## Step 7: Restart Apache

```
sudo systemctl restart apache2.service
```

and you're all set to go.

Head to `http://example.com` and you'll see `Laravel` homepage.

[1]: https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-ubuntu-18-04 "How To Install Linux, Apache, MySQL, PHP (LAMP) stack on Ubuntu 18.04"
