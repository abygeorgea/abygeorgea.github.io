---
layout: post
title: "Migrating To Octopress"
date: 2017-05-20 05:57:24 +1000
comments: true
categories: 
- Octopress
- Octostrap3
---

Over the past weekend, I noticed that my blog is not available since azure has disabled hosting of my WordPress blog. It happened because I ran out of my free credits for the current month. I started looking for alternate options for hosting WordPress. That's when I came across ([Static Generator is All a Blog Needs - Moving to Octopress](http://www.rahulpnath.com/blog/static-generator-is-all-a-blog-needs-moving-to-octopress/)). I decided to give it a try. 

Below are the main steps which I followed for migrating to Octopress
# Documentation #

-  Read documentation of Octopress [here](http://octopress.org/docs/) and Jekyll [here](https://jekyllrb.com/docs/home/)

# Setup #

- Install Chocolatey as mentioned in documentation [here](https://chocolatey.org/install)
Below command can be run on cmd.exe open as administrator
``` plain cmd.exe
@powershell -NoProfile -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```

-  As mentioned in octopress documentation, ensure Git, ruby and devkit are installed. Cholocatey way of installation can be found in [git](https://chocolatey.org/packages/git.install), [ruby](https://chocolatey.org/packages/ruby) , [devkit](https://chocolatey.org/packages/ruby2.devkit).
Below commands can be run on cmd.exe
``` plain cmd.exe
choco install git.install
choco install ruby
choco install ruby2.devkit
```

-  By default, devkit is installed in `C:\tools\`. Move in `devkit` folder and run below commands
``` plain cmd.exe
ruby dk.rb init
ruby dk.rb install
gem install bundler
```
# Install Octopress #

-  Now install Octopress as per [documentation](http://octopress.org/docs/setup/)
``` plain cmd.exe
git clone git://github.com/imathis/octopress.git octopress
cd octopress
bundle install
rake install // Install default Octopress theme
```

# Install Octostrap3 theme & Customize#

- Since I didn't like the default theme much, I installed Octostrap3 theme as mentioned [here](http://kaworu.github.io/octostrap3/setup/install/)
``` plain cmd.exe
git clone https://github.com/kAworu/octostrap3.git .themes/octostrap3
rake "install[octostrap3]"
```
- Fix up all issues. The date displayed as "Ordinal" can be fixed by updating `_config.yml` file as mentioned in their blog. Below is the config which I used

``` plain _config.yml
date_format: "%e %b, %Y"
```
- I made few more changes for changing the navigation header color, color of code blocks and also to include a side bar with categories. The changes are as below

Changing color of code blocks is done by commenting below line in `octopress\sass\custom\_colors.scss`
``` plain
\\$solarized: light;
```

Navigation header color is changed by adding below to `octopress\sass\custom\_styles.scss`
```
.navbar-default {
    background-image: -webkit-gradient(linear,left top,left bottom,from(#263347),to(#263347));
}
.navbar-default .navbar-brand {
    color: #fff;
}
.navbar-default .navbar-nav>li>a {
    color: #fff;
}
```

Adding category side bar is done by following steps mentioned in [Category List Aside](https://kaworu.github.io/octostrap3/blog/2013/10/03/category-list-aside/)

# Google Analytics Integration#
Next step was google analytics integration. Detailed steps for this is available on various blogs. Below is what I followed

- Sign up for google analytics ID in [here](https://analytics.google.com/analytics/web/provision?authuser=0#provision/SignUp/)
- Update _config.yml with google analytics ID
```
# Google Analytics
google_analytics_tracking_id: UA-XXXXXXXX-1
```
- Update `google_analytics.html` file with below

```
   <script>
    (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
    (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
    m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
    })(window,document,'script','//www.google-analytics.com/analytics.js','ga');

    ga('create', 'UA-XXXXXXXX-1', 'auto');
    ga('send', 'pageview');

  </script>
```
- UA-XXXXXXXX-1 can be replaced with `site.google_analytics_tracking_id` enclosed in double braces/curly brackets
- Log in to Google Analytics site and navigate to Admin >> View >> Filters
- Add a new filter to exclude all traffice to hostname "localhost". This will help to exclude all site visit done for development/ preview purpose.



# Sample Post #

-  Now create a Hello World post and check how it look

``` plain cmd.exe
rake new_post["Hello World"]
rake generate
rake preview
```
rake preview mounts a webserver at [http://localhost:4000](http://localhost:4000 "http://localhost:4000"). By opening a browser window and navigating to [http://localhost:4000](http://localhost:4000 "http://localhost:4000") will preview the Hello World Post


# Deploying to GitHub Pages #

Detailed instructions can be found in [Deploying to Github Pages](http://octopress.org/docs/deploying/github/). Below are high-level steps copied from there
- Create a GitHub repository with name `yourusername.github.io`
- Run below command. It will prompt for GitHub URL, which needs to be filled in
```
rake setup_github_pages // This does all configurations
rake generate
rake deploy
```
- Now we can commit the source 
```
git add .
git commit -m 'your message'
git push origin source
```

# Custom Domain #
- Create a file named `CNAME` in blog source
- Update it with custom domain name. It has to be a sub domain (www.examplesubdomain.com)
- Update the CNAME dns setting in your domain provider to point to `https://username.github.io`
- If top-level domains (exampletopdomain.com) are needed, then configure A record to point to IP address `192.30.252.153` or `192.30.252.154`.


# Migrating Old blog Post from word press #

After completing above steps,  a new octopress blog is ready to go . Below are the steps which I followed to migrate old blog posts from word press.



- Clone [Exitwp](https://github.com/thomasf/exitwp)
- Follow the steps mentioned in readme.md. 
    - Export old wordpress blog using WordPress exporter in tools/export in WordPress admin
    - Copy xml file to wordpress-xml directory
    - Run  python exitwp.py in the console from the same directory of unzipped archive
    - All blogs will be created as separate directory under `build` directory
    - Copy relevant folders to source folder of the blog

- Find broken redirection links and fix
- Find broken image links and fix