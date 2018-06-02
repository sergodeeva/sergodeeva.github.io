---
layout: post
title:  "How to create a blog using Hexo on GitHub Pages"
date:   2017-12-16 17:01:36 +0800
tags: hexo githubpages
description: "Step by step instruction on creating a personal website using Hexo framework"
image: "/assets/post_images/2017-12-16-hexo.png"
---

I was originally planning to create a blog using [Create React App](https://github.com/facebookincubator/create-react-app). It was extremely easy to set up `create-react-app`, but "the devil is in the details". So I decided to use Hexo framework instead, as did not want to spend much time googling stuff and figuring out the details.

Before I started, I did not have any experience with `hexo`, and had no idea about `node` and `nvm`. So below an instruction for beginners :)

## Install Git
Install Git if you don't have it yet ([installation instructions](https://www.atlassian.com/git/tutorials/install-git)).

## Install nvm (Node Version Manager)
{% highlight terminal %}
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash
{% endhighlight %}

If you get `nvm: command not found` after running the install script, try solutions described [here](https://github.com/creationix/nvm#installation). In my case I solved the problem with `nvm` by adding this line of code into `~/.bash_profile`file:

{% highlight terminal %}
source ~/.bashrc
{% endhighlight %}

## Install Node.js
Hexo framework is powered by `node.js`, that is why you need to have `node` installed before proceeding with Hexo installation.

{% highlight terminal %}
nvm install node
{% endhighlight %}

## Install Hexo
{% highlight terminal %}
npm install -g hexo-cli
{% endhighlight %}

## Create blog
Suppose you want to create your blog in a local folder `my-blog`. Run one by one the following commands:

{% highlight terminal %}
hexo init my-blog
cd my-blog
hexo generate # to generate the blog
hexo server # to start hexo server
{% endhighlight %}

Voila! The blog is ready, check it out at http://localhost:4000/.

## Configure blog settings
Now you can start configuring your blog using `_config.yml` file in the root of your blog's folder. There are lots of settings can be configured ([read more about it](https://hexo.io/docs/configuration.html)). To start with, you can configure some personal information as did I:

{% highlight yaml %}
# Site
title: Natalya Kosenko's Blog
subtitle:
description:
author: Natalya Kosenko
language: en
timezone: Asia/Singapore
{% endhighlight %}

## Change theme
Now it is time to beautify your blog. You can choose one of a few dozens free themes available [here](https://hexo.io/themes/). My choice was `cactus-white` that is based on a [cactus-dark](https://github.com/probberechts/cactus-dark) theme. Below are a few commands you need to run one by one in the Terminal in order to switch theme from default one to the `cactus-white`:

{% highlight terminal %}
git clone git@github.com:sergodeeva/cactus-white.git themes/cactus-white
npm install hexo-pagination --save
{% endhighlight %}

After you installed a new theme, you need to change the `theme` property in the `_config.yml` file:

{% highlight yaml %}
# theme: landscape
theme: cactus-white
{% endhighlight %}

To see the changes at http://localhost:4000/, you need to re-generate blog by running `hexo generate`, and then restart the hexo server (stop it and then run `hexo server`).

## Write a new post
To create a new post or a new page, you can use the following commands:

{% highlight terminal %}
hexo new post <title> # to create a new post
hexo new page <title> # to create a new page
hexo new draft <title> # to create a new draft
hexo new post "Hello world" # example
{% endhighlight %}

A file with .md extension will be created in a `source` folder. You can write your post using [markdown](https://daringfireball.net/projects/markdown/syntax) - lightweight markup language with plain text formatting syntax.

## Deploy to GitHub Pages  
Make sure you have created a GitHub repository with a name `username.github.io`. For example, my repository is `sergodeeva.github.io`.
Install hexo deployer plugin:

{% highlight terminal %}
npm install hexo-deployer-git --save
{% endhighlight %}

Once the plugin is installed, change deployment settings in the `_config.yml`file (replace 'username' with your GitHub username):

{% highlight yaml %}
deploy:
  type: git
  repo: git@github.com:username/username.github.io.git
  branch: master
{% endhighlight %}

Deploy the blog to GitHub pages using this command:
{% highlight terminal %}
hexo d -g # shortcut for `hexo generate` and `hexo deploy`
{% endhighlight %}

Done, your blog is now available at `http://username.github.io` page.

The whole process took me a few hours (90% of that time I spent choosing a blog theme).