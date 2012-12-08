---
layout: post
title: "Setup Octopress on Github and Heroku"
date: 2012-12-08 14:18
comments: true
categories: Octopress 
---

## Introduction

__Octopress__ is a fantasic blogging framework which supports Github, Heroku. It's very esay to set it up, within 5 minutes, a powful and free blog is ready for you. Go ahead and get your hands dirty, nothing gonna be lost!

## Installation

Octopress requires Ruby 1.9.3, so make sure Ruby 1.9.3 has already been installed on you computer. RVM is recommended to install Ruby if Ruby 1.9.3 is missing.

```
$ ruby --version  # Should report Ruby 1.9.3
# If not use below command to install
$ rvm install 1.9.3
```

### 1. Get Octopress
You have to get the Octopress from Github.

```
$ cd path/to/directory_where_you_want_to_install_octopress
$ git clone git://github.com/imathis/octopress.git octopress 
```

### 2. Install Dependencies
Octopress relies on several gems, which should be installed before configuration and running. It's great that Octopress uses Bundler to manage the dependency, so just use few commands to finish configuration.

You could create a gemset for Octopress if you are using RVM.

```
$ rvm --create use 1.9.3@octopress
```

And then replace the contents of the `.rvmrc` file under the root directory of octopress folder with:

```
rvm use 1.9.3@octopress
```

Then install gems:

```
$ gem install bundler # If Bundler is not installed otherwise ignore this command
$ bundle install # This command will install all required gems automatically
```

### 3. Setup Octopress

You could install default Octopress theme by below command:

```
$ cd path/to/path_to_octopress_directory
$ rake install
```

### 4. Custom Configuration

You only need to edit `_config.yml` to finish configuration. You’ll want to change `title`, `subtitle` and `author` and if you decide to use your own domain, you also have to change `url`. Below are guides from [official document](http://octopress.org/docs/configuring/).

```
url:                # For rewriting urls for RSS, etc
title:              # Used in the header and title tags
subtitle:           # A description used in the header
author:             # Your name, for RSS, Copyright, Metadata
simple_search:      # Search engine for simple site search
description:        # A default meta description for your site
date_format:        # Format dates using Ruby's date strftime syntax
subscribe_rss:      # Url for your blog's feed, defauts to /atom.xml
subscribe_email:    # Url to subscribe by email (service required)
category_feeds:     # Enable per category RSS feeds (defaults to false in 2.1)
email:              # Email address for the RSS feed if you want it.
```

To use `Kramdown` as default `Markdown` parser, instead of `RDiscount`, change `markdown: rdiscount` to `kramdown`.

### 5. Preparation for Deploy

I'd like to use Github to host my blog, so below are steps on how to deploy Octopress to Github. Please refer to the [offical document](http://octopress.org/docs/deploying/) if you prefer Heroku of Rsync or visit [Moncef Belyamani ](http://www.moncefbelyamani.com/how-to-install-and-configure-octopress-on-a-mac/) if you think Amazon S3 is good choice.

As we want to deploy to Github, you need a Github account. If you still don't have, it's time to get one. Please add the ssh key (which could be found at ~/.ssh/id_rsa.pub) to your Github `Account Settings>SSH Keys` to make your life easier.

#### 5.1 Create repo for Octopress

Create a new [Github repository](https://github.com/repositories/new) with any name you likei, I choose `blog` here. We'll take advantage of Github’s Project Pages service which allows you to host a site for your existing open source project. Github will look for a gh-pages branch in your project’s repository and make the contents available at url like `http://username.github.com/project`.

Here is the command help you set up Octopress site to publish to your projects gh-pages repository:

```
$ rake setup_github_pages
```

Then you'll be asked to provide repository url, input `git@github.com:username/blog.git`. 

Next run:

```
# Commit configuration changes
$ git add .
$ git commit -m 'Custom configurations'

# Deploy Octopress to Github
$ rake generate
$ rake deploy
```

One minute later, please go to `http://username.github.com/blog` to enjoy you new blog. Quite easy, isn't it!

## Writing Blogs

Octopress provides a rake task to create new blog posts with the right naming conventions, with sensible yaml metadata.

#### Syntax

```
$ rake new_post["title"]
```

`new_post` expects a naturally written title and strips out undesirable url characters when creating the filename. The default file extension for new posts is `markdown` but you can configure that in the `Rakefile`.

#### Example 

```
rake new_post["Zombie Ninjas Attack: A survivor's retrospective"]
# Creates source/_posts/2011-07-03-zombie-ninjas-attack-a-survivors-retrospective.markdown
```

Blog posts will be stored in the `source/_posts` directory with naming conventions: `YYYY-MM-DD-post-title.markdown`.

The filename will determine your url. With the default permalink settings the url would be something like `http://site.com/blog/2011/07/03/zombie-ninjas-attack-a-survivors-retrospective/index.html`.

### 1. Write Your First Blog

Edit the `source/_posts/2011-07-03-zombie-ninjas-attack-a-survivors-retrospective.markdown` file with your favorite editor. Then you’ll see a block of yaml at the begnning which decides how to processes posts and pages. 

```
---
layout: post
title: "Zombie Ninjas Attack: A survivor's retrospective"
date: 2011-07-03 5:59
# turn comments on/off
comments: true
#
# Multiple categories
# categories: [CSS3, Sass, Media Queries]
# or
# categories:
# - CSS3
# - Sass
# - Media Queries
categories: Octopress
# Set to false if you are working on draft
published: true
---
```

Then you could add your content below the header. Please refer to [`http://octopress.org/docs/`](http://octopress.org/docs) for the usage of tags. 

### 2. Preview Post

Once you content is written you could preview your post locally.

```
$ rake generate   # Generates posts and pages into the public directory
$ rake preview    # Watches, and mounts a webserver at http://localhost:4000
```

If you set `published: false` in the `YAML` header, you could preview the post with `rake preview`, but it won’t get published by `rake generate`.

### 3.1 Deploy to Github

If you are satify with the post, you could deploy the new post to Github server:

```
$ rake deploy
```

### 3.2 Deploy to Heroku

Heroku is a good choice to host blog too. Again, you need an Heroku account, then install the Heroku tool.

```
$ gem install heroku
$ heroku login # Login first and you won't be asked for password during deploying.
``` 

Next we create an app on heroku that will host the blog

```
$ heroku apps:create app_name
$ git remote add heroku git@heroku.com:app_name.git # Add remote
```

To deploy to Heroku, you have to add the `public` folder to git repo, which keep all compiled files. Remove the `public` line from `.gitignore` file. And commit the content in the foler. Please remember you always have to commit all the changes in the folder before deploying to Heroku.

```
$ rake generate
$ git add .
$ git commit 'Add posts for Heroku'
$ git push heroku master
```

Then you could find you blog at `http://app_name.herokuapp.com/blog`. If you don't want to input the blog every time, please refer to the `Use Public Folder as ROOT` section of `Custom Octopress` below.

## Custom Octopress

As a blogger, we always hope our tools could be more powerful. Octopress is easy to be customed to satisfy special requirement.

### 1. Add Table Support

The standard `Markdown` syntax is not support `table`, however there are several extentions. `Kramdown` is support `table` very much. However the CSS style of Octopress make the border of table invisible. We should do some hacks to make the table nicer. [1](http://samwize.com/2012/09/24/octopress-table-stylesheet/), [2](http://programus.github.com/blog/2012/03/07/add-table-data-css-for-octopress/)

#### Step 1 

Create `source/stylesheets/table.css` and add below content.

{% codeblock table.css %}
* + table {
  border-style:solid;
  border-width:1px;
  border-color:#e7e3e7;
}

* + table th, * + table td {
  border-style:dashed;
  border-width:1px;
  border-color:#e7e3e7;
  padding-left: 3px;
  padding-right: 3px;
}

* + table th {
  border-style:solid;
  font-weight:bold;
  background: url("/images/noise.png?1330434582") repeat scroll left top #F7F3F7;
}

* + table th[align="left"], * + table td[align="left"] {
  text-align:left;
}

* + table th[align="right"], * + table td[align="right"] {
  text-align:right;
}

* + table th[align="center"], * + table td[align="center"] {
  text-align:center;
}
{% endcodeblock %}

#### Step 2

Edit `source/_includes/head.html`, add:

{% codeblock %}
  <link href="{{ root_url }}/stylesheets/table.css" media="screen, projection" rel="stylesheet" type="text/css" />
{% endcodeblock %}

after line:

```
 <link href="{{ root_url }}/stylesheets/screen.css" media="screen, projection" rel="stylesheet" type="text/css">
```

Then you can see:

```
|--------------+---------------+--------------|
| First Header | Second Header | Third Header |
| :------------| :-----------: | ------------:|
| Cell         |      Cell     |         Cell |
|--------------+---------------+---------------
| Cell         |      Cell     |         Cell |
|--------------+---------------+--------------|
```

|--------------+---------------+--------------|
| First Header | Second Header | Third Header |
| :------------| :-----------: | ------------:|
| Cell         |      Cell     |         Cell |
|--------------+---------------+---------------
| Cell         |      Cell     |         Cell |
|--------------+---------------+--------------|


### 2. Use Public folder as ROOT

It's quite easy, all you have to do is edit the `_config.yml` and regenerate files.

{% codeblock _config.yml %}
-subscribe_rss: /blog/atom.xml
+subscribe_rss: /atom.xml
-root: /blog
+root: /
-permalink: /blog/:year/:month/:day/:title/
+permalink: /:year/:month/:day/:title/
-destination: public/blog
+destination: public/
-category_dir: blog/categories
+category_dir: categories
-pagination_dir: blog  # Directory base for pagination URLs eg. /blog/page/2/
+pagination_dir: /  # Directory base for pagination URLs eg. /blog/page/2/
{% endcodeblock %}

Then regenerate files:

```
$ rake generate
$ rake preview
```

And you could find your blog at `http://127.0.0.1:4000/` or `app_name.herokuapp.com` if it has been deployed to Heroku. 

