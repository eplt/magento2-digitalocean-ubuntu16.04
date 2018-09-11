[![Build Status](https://travis-ci.org/magento/magento2.svg?branch=develop)](https://travis-ci.org/magento/magento2)
[![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/magento/magento2?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)
[![Crowdin](https://d322cqt584bo4o.cloudfront.net/magento-2/localized.png)](https://crowdin.com/project/magento-2)
<h2>Welcome</h2>
Welcome to Magento 2 installation! This repo is for installation of Magento 2 on DigitalOcean running Ubuntu 16.04. For more information about Magento 2, take a look at the original repo here. [https://github.com/magento/magento2](https://github.com/magento/magento2) - Retested on Version 2.2.5 and still the LAMP droplet on 16.04.


## Basic requirements
* [Magento system requirements](http://devdocs.magento.com/guides/v2.2/install-gde/system-requirements2.html), you can check this if you want, but you will get all the needed components running in your droplet if you follow this guide.
* [DigitalOcean Account - Click here for $10 referral credit](https://m.do.co/t/b298d6966c0c), you will need a DigitalOcean account with a new one-click LAMP droplet running on Ubuntu 16.04, I have included my referral link, you can get 10 bucks which is enough for 2 months of the cheapest droplet, or 1 month of the 2GB-RAM droplet which is recommended for running Magento smoothly.
* [Magento Account](https://magento.com/tech-resources/download), you can use this repo to clone the code for installation, but it might be easier if you setup an account at Magento.com to download other stable versions, some with sample data, for your installation. 
* [SFTP client](https://filezilla-project.org/), if you decide to download the zip files above, you will need SFTP client to upload it to your droplet.
* Domain name, for setting up SSL using Let's Encrypt. You might be able to use some DDNS services with it as well.

## Step 1 - Get your Ubuntu 16.04 Ubuntu Droplet
Get your droplet ready, log in with SSH, note down the passwords and the mysql root password from the fresh install.

## Step 2 - Set locale and run apt update/upgrade

```
pico /etc/default/locale
```

Add following lines to locale file:
```
LANG="en_US.UTF-8"
LANGUAGE="en_US:en"
LC_ALL="en_US.UTF-8"
```
Then run:
```
sudo apt-get update
sudo apt-get upgrade
```
## Step 3 - Setup Magento site

```
sudo nano /etc/apache2/sites-available/magento.conf
```
Add the following lines:
```
<VirtualHost *:80>
    DocumentRoot /var/www/html
    <Directory /var/www/html/>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
    </Directory>
</VirtualHost>
```
Then run:
```
sudo a2ensite magento.conf
sudo a2dissite 000-default.conf
sudo a2enmod rewrite
sudo service apache2 restart
```

## Step 4 - Setup PHP

```
sudo nano /etc/php/7.0/apache2/php.ini
```

edit this line to increase memory limit
```
memory_limit = 512M
```
then run these:
```
sudo apt-get install php7.0-curl php7.0-gd php7.0-mcrypt php-xml php7.0-soap php7.0-bcmath php7.0-intl php7.0-zip php7.0-mbstring
sudo phpenmod mcrypt
```

## Step 5 - Setup MySQL

Start MySQL
```
mysql -u root -p
```
Run the following in MySQL to create database and user, choose a <password> properly.

```
CREATE DATABASE magento;
CREATE USER magento_user@localhost IDENTIFIED BY '<password>';
GRANT ALL PRIVILEGES ON magento.* TO magento_user@localhost IDENTIFIED BY '<password>';
FLUSH PRIVILEGES;
exit
```

## Step 6 - Install the Magento files
Download the Magento file archive to your home directory manually from Magento.com
In my case, I downloaded Magento-CE-2.2.2_sample_data-2017-12-11-09-37-31.tar.gz.

```
mv Magento-CE-2.2.2_sample_data-2017-12-11-09-37-31.tar.gz /var/www/html/
cd /var/www/html/
tar xzvf Magento-CE-2.2.2_sample_data-2017-12-11-09-37-31.tar.gz
rm Magento-CE-2.2.2_sample_data-2017-12-11-09-37-31.tar.gz
sudo chown -R www-data:www-data /var/www/html/
```

## Step 7 - Setup Let's Encrypt
You need to have mapped your domain name to the IP address.
Follow the setup screens below, I suggest not setting up for force all HTTPS traffic, you can do that inside Magento setup later.

```
cd /usr/local/sbin
sudo wget https://dl.eff.org/certbot-auto
sudo chmod a+x /usr/local/sbin/certbot-auto
certbot-auto --apache -d <domain name, without https:// part, for example, www.github.com>
```
You can check your certificate using the following URL.
https://www.ssllabs.com/ssltest/analyze.html?d=<domain name, without https:// part, for example, www.github.com>&latest

## Step 8 - Setup Cron Jobs for indexes and auto Let's Encrypt renewal

```
sudo crontab -e
```
Add the following lines:

```
30 2 * * 1 /usr/local/sbin/certbot-auto renew >> /var/log/le-renew.log
* * * * * /usr/bin/php /var/www/html/bin/magento cron:run | grep -v "Ran jobs by schedule" >> /var/log/magento.cron.log
* * * * * /usr/bin/php /var/www/html/update/cron.php >> /var/log/update.cron.log
* * * * * /usr/bin/php /var/www/html/bin/magento setup:cron:run >> /var/log/setup.cron.log
```

## Step 9 - Restart Apache

```
service apache2 restart
```

## Step 10 - Web setup of Magento - Well Done!
Visit your new domain for the first time to get started.
You will need the username and password you setup in MySQL for the database user access.
It should now be working. Well done!

## Other Useful Commands

To reindex Magento
```
php /var/www/html/bin/magento indexer:reindex
```

Check your php path
```
which php
```

## Other Installation Documents
*	[Magento DevBox](https://magento.com/tech-resources/download), the easiest way to get started with Magento.
*	[Installation guide](http://devdocs.magento.com/guides/v2.2/install-gde/bk-install-guide.html)

<h2>License</h2>

Each Magento source file included in this distribution is licensed under OSL 3.0 or the Magento Enterprise Edition (MEE) license

http://opensource.org/licenses/osl-3.0.php  Open Software License (OSL 3.0)
Please see LICENSE.txt for the full text of the OSL 3.0 license or contact license@magentocommerce.com for a copy.

Subject to Licensee's payment of fees and compliance with the terms and conditions of the MEE License, the MEE License supersedes the OSL 3.0 license for each source file.
Please see LICENSE_EE.txt for the full text of the MEE License or visit http://magento.com/legal/terms/enterprise.

