OpenPhoto / Installation for Ubuntu + Apache
=======================
#### OpenPhoto, a photo service for the masses

## OS: Linux Ubuntu Server 10.04+

This guide instructs you on how to install OpenPhoto on an Ubuntu Server.

----------------------------------------

### Prerequisites

#### Database and File System Options

##### MySql 
You'll need to provide credentials for a MySql database. If the database doesn't already exist it will be created. If the user doesn't have `CREATE DATABASE` permissions then make sure it's already created.

##### AWS
If you're going to use AWS S3 then You'll need to be [signed up it](http://aws.amazon.com/s3/)

#### Server Packages and Modules
Once you've confirmed that your cloud account is setup you can get started on your server. For that you'll need to have _Apache_, _PHP_ and _curl_ installed with a few modules.

    apt-get update
    apt-get upgrade
    apt-get install apache2 php5 libapache2-mod-php5 php5-mysql php5-curl php5-gd php5-mcrypt php-apc build-essential libpcre3-dev php-pear
    a2enmod rewrite
    
    #### Bug in Ubuntu 14.04 where the mcrypt module isn't loaded for PHP
    #### Requires you to run the following if you see this in your error logs
    ####   Call to undefined function mcrypt_create_iv()
    ln -s /etc/php5/mods-available/mcrypt.ini /etc/php5/apache2/conf.d/20-mcrypt.ini


There are also a few optional but recommended packages and modules.

    apt-get install php5-dev php5-imagick exiftran
    pecl install oauth
    a2enmod deflate
    a2enmod expires
    a2enmod headers

----------------------------------------

### Installing OpenPhoto

Download and install the source code. We recommend `/var/www/yourdomain.com` but you can use any directory you'd like.

#### Using git clone

    apt-get install git-core
    git clone git://github.com/photo/frontend.git /var/www/yourdomain.com

#### Using tar

    cd /var/www
    wget https://github.com/photo/frontend/tarball/master -O openphoto.tar.gz
    tar -zxvf openphoto.tar.gz
    mv openphoto-frontend-* yourdomain.com

Assuming that this is a development machine you only need to make the config writable by the user Apache runs as. Most likely `www-data`.

    mkdir /var/www/yourdomain.com/src/userdata
    mkdir /var/www/yourdomain.com/src/html/photos
    mkdir /var/www/yourdomain.com/src/html/assets/cache
    chown www-data:www-data /var/www/yourdomain.com/src/userdata
    chown www-data:www-data /var/www/yourdomain.com/src/html/photos
    chown www-data:www-data /var/www/yourdomain.com/src/html/assets/cache

----------------------------------------

### Setting up Apache and PHP

#### Apache

Copy the sample virtual host configuration file. Replace yourdomain with your own domain name:

    cp /var/www/yourdomain.com/src/configs/openphoto-vhost.conf /etc/apache2/sites-available/yourdomain.conf

Edit /etc/apache2/sites-available/yourdomain.conf and replace instances of `/path/to/openphoto/html/directory` with `/var/www/yourdomain.com/src/html` or wherever you placed the code. Depending on which modules you installed you may have to comment out some of the `Expires*` directives.

    vi /etc/apache2/sites-available/yourdomain.conf
    
By default, any access to ini files is denied with a "Not Found" 404 HTTP code.  To enable a 403, or Forbidden return code, change the following lines in the virtual host file.

Uncomment:

    # 403 Forbidden for ini files
    #RewriteRule \.ini$ - [F,NC]

Comment:

    # 404 Not Found for ini files
    AliasMatch \.ini$	/404

Enable openphoto:

    a2ensite yourdomain.conf
    
You may also want to disable Apache's default virtual host (especially if you are not running your website on your local machine):

    a2dissite default.conf
    
If you want to run your website on your local machine, don't forget to modify your hosts file:

    vi /etc/hosts

Add:
    127.0.1.1 yourdomain.com
    
### PHP

Edit your `php.ini` file:

    vi /etc/php5/apache2/php.ini

Search for the following values and make sure they're correct.

    file_uploads = On
    upload_max_filesize = 16M
    post_max_size = 16M
    
Search for, and if needed add the following line to load the Oauth Extention.

    extension=oauth.so

Restart apache:

    /etc/init.d/apache2 restart

### Launching your OpenPhoto site

Now you're ready to launch your OpenPhoto site. Point your browser to yourdomain.com and you'll be taken to a setup screen.

If you're using a cloud service you'll need those account credentials to continue.

Once you complete the 3 steps your site will be up and running and you'll be redirected there. The _setup_ screen won't show up anymore. If for any reason you want to go through the setup again you will need to delete the generated config file and refresh your browser.

    rm /var/www/yourdomain.com/src/userdata/configs/yourdomain.com.ini

**ENJOY!**

