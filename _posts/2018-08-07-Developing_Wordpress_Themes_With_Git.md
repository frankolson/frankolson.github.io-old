---
layout:     post
title:      "Developing Wordpress Themes with Git"
date:       2018-08-07 12:00:00
author:     "Will Olson"
header-img: "img/post-bg-code.jpg"
---

As a Ruby on Rails developer, I love git. I commit almost as much as I save my work (I'm surprised the `s` key on my keyboard hasn't worn out yet). So it came as a surprise to me that I could not find a significant amount of resources on the web for developing Wordpress themes using Git and Github.

Not to worry though, after a bit of Googling and some experimentation I found a system that works for me. I use a MacBook Pro, but the steps outlined here should work for Linux users as well (sorry Windows fans).

Here we go!

## Local Development

First, I like to organize my home directory by project type, so we are going to create two folders to store our Wordpress sites and Wordpress themes.

```shell
cd ~/
mkdir Wordpress
mkdir WpThemes
```

Also, we are going to want to have [MAMP installed and configured for Wordpress](https://skillcrush.com/2015/04/14/install-wordpress-mac/). Once that is done we should create a new local Wordpress site. I'm going to be super original and call mine "CustomSite".

Once that is done of we should create a basic theme in our new `WpThemes` folder:

```shell
cd ~/WpThemes
mkdir custom_theme
cd custom_theme
touch index.php style.css
```

Next, lets added the needed comments to `style.css` and let Wordpress know this is a theme:

```
/*
Theme Name: Custom Theme
Author: Will Olson
Description: A custom template for my new Wordpress Site
Version: 0.0.1
*/

```

The final step needed to get Wordpress to recognize our new theme is to create a symlink between our theme folder and our local Wordpress site:

```shell
cd ~/Wordpress/CustomSite/wp-content/themes
link -s custom_theme ~/WpThemes/custom_theme
```

Great! Now if you boot up your MAMP server and open your browser to [localhost:8888/CustomSite/admin](localhost:8888/CustomSite/admin), you should be able to log in and activate your new custom theme!

## GitHub Hosting

I'm going to keep this brief as using GitHub is pretty easy.

First you should [create a Github repository](https://help.github.com/articles/create-a-repo/) to store your code, I'm going to call mine "custom_theme" (again, I realize this is super original). Once that is taken care of I am going to initialize a new git repository locally and push my code up to my new GitHub Repository.

```shell
cd ~/WpThemes/custom_theme
git init
git add .
git commit -m "Initial commit"
git remote add origin [github_repository_url]
git push -u origin master
```

And that's it.

## Deployment

Now, its time to get our theme to a live Wordpress Site. I am going to work under the assumption that you have shell access to your server. If you haven't set up a server yet, I would hightly recommend using [DigitalOcean's Wordpress One-Click Install](https://www.digitalocean.com/community/tutorials/how-to-use-the-wordpress-one-click-install-on-digitalocean). They are resonably priced and have simple and easy to use products.

First thing, let's shell into our server and find our Worpress files (most of the time they will be in `/var/www/html`):

```shell
ssh [user]@[server_address]
cd /var/www/html
cd /wp-content/themes
```

Finally, we are going to clone our theme files from our GitHub repository:

```shell
git clone [github_repository_url]
```

Our theme should show up in our admin dashboard, and we are done!

Now any time we want to update our theme files we just have to:

1. Develop and commit our changes locally:

   ```shell
   git add .
   git commit -m "Awesome new changes"
   ```

2. Push our changes to our GitHub repository:

   ```shell
   git push origin master
   ```

3. Log into our server and pull down the changes:

   ```shell
   ssh [user]@[server_address]
   cd /var/www/html/wp-content/themes/custom_theme
   git pull origin master
   ```

## In Review

1. Create git repository to store our theme code locally.
2. Symlink it to our local Wordpress site and develop the theme code.
3. Add the theme code to a GitHub repository.
4. Clone the GitHub repository to the themes folder of your live Wordpress site.
