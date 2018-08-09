---
layout:     post
title:      "Local Wordpress Development Setup on a Mac"
date:       2018-08-09 12:00:00
author:     "Will Olson"
---

I find myself creating a lot of Wordpress sites locally while developing for clients, so I thought I'd create quick list of steps to jog my memory in the future. Hope it helps!

### 1. Create a `Wordpress` directory in your `home` directory

```shell
cd ~/
mkdir Wordpress
```

### 2. Install and configure MAMP

Got to the [Mamp downloads page](https://www.mamp.info/en/downloads/) and download the latest release of MAMP for Apple:

![MAMP download page]({{ site.baseurl }}/img/local_wordpress_install/mamp_download_page.png)

Then, run the installer downloaded:

![MAMP installer]({{ site.baseurl }}/img/local_wordpress_install/mamp_installer.png)

Once that is completed, we need to configure MAMP to look at our recently created `Wordpress` directory. To do that, launch MAMP and open its preferences with `cmd + ,`. Then click "Web Server" in the navigation bar and next click "Show in Finder".

![MAMP configuration]({{ site.baseurl }}/img/local_wordpress_install/mamp_configuration.png)

This will open a Finder window where you will select the `Wordpress` directory we created earlier. After that is selected and you are back at the MAMP configuration window, click "OK" and we will move on to the next steps.

### 3. Create a new Wordpress site in your `Wordpress` directory

```shell
cd ~/Downloads
curl -O https://wordpress.org/latest.zip
unzip latest.zip
mv wordpress ~/Wordpress/site_name
rm latest.zip
```

### 4. Start MAMP and create a new Wordpress database

Start your MAMP server

![MAMP start screen]({{ site.baseurl }}/img/local_wordpress_install/mamp_start_screen.png)

A web page will launch in your browser and you should click the [phpMyAdmin link](http://localhost:8888/MAMP/phpmyadmin.php?lang=en).

![MAMP browser start page]({{ site.baseurl }}/img/local_wordpress_install/mamp_browser_start_page.png)

Click on "New" link in the site bar to create go to the database creation form.

![phpMyAdmin start page]({{ site.baseurl }}/img/local_wordpress_install/phpmyadmin_start_page.png)

And name a database after you project (not necessary, but a convention I like to follow).

![phpMyAdmin new database form]({{ site.baseurl }}/img/local_wordpress_install/phpmyadmin_new_database_form.png)

### 5. Go to your new local site and complete the Wordpress setup

Go to your browser and visit [localhost:8888/site_name](localhost:8888/site_name), where you will be able to complete your setup.

![Wordpress setup start page]({{ site.baseurl }}/img/local_wordpress_install/wordpress_setup_start.png)

When filling in your database credentials you should use the credentials on the page that launched when you started your MAMP server:

![MAMP database form]({{ site.baseurl }}/img/local_wordpress_install/mamp_database_credentials.png)

![Wordpress database credentials form]({{ site.baseurl }}/img/local_wordpress_install/wordpress_database_credentials.png)

Voila! You're done.

## In Summary

After your first time running through these steps, in the future you only need to perform Steps 3-5.

1. Create a `Wordpress` directory in your `home` directory
2. Install and configure MAMP
3. Create a new Wordpress site in your `Wordpress` directory
4. Start MAMP and create a new Wordpress database
5. Go to your new local site and complete the Wordpress setup
