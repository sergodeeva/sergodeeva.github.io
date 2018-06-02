---
layout: post
title:  "How to switch Jekyll theme on GitHub Pages site"
date:   2017-12-23 14:04:12 +0800
tags: jekyll githubpages
description: "Changing theme of the existing Jekyll installation."
image: "/assets/post_images/2017-12-23-jekyll.png"
---

A new Jekyll site out of the box uses a default gem-based theme called `Minima`. However you can choose from one of many custom themes supported by GitHub Pages: [official Jekyll themes](https://pages.github.com/themes/), as well as [open source Jekyll themes](https://github.com/topics/jekyll-theme) hosted on GitHub.

I was recently switching a theme for my blog from `Minima` to `Cayman` and found it not very obvious. Although there are a lot of instructions available on the Internet, I spent a couple of hours dealing with various errors during the switch. Thus I decided to write a post about switching Jekyll theme, hope it will help you to save some time.

## Prerequisites
Make sure you have `Jekyll` installed (if not, follow the [installation instructions](https://jekyllrb.com/docs/installation/)).
As we are going to use GitHub Pages, make sure you have created a GitHub repository with a name `username.github.io`. For example, my repository is `sergodeeva.github.io`.

## Update gems
Navigate to `Gemfile` in the root directory of your `Jekyll` site, remove gem `jekyll` and add `github-pages` gem:
{% highlight ruby %}
# gem "jekyll", "~> 3.6.2"
gem "github-pages", group: :jekyll_plugins
{% endhighlight %}

Run `bundle update` command in Terminal to update all gems.

## Create layouts
Default Jekyll theme `Minima` has 4 layouts that are used on different pages of your blog:
* `default.html` - The base layout that lays the foundation for subsequent layouts.
* `home.html` — The layout for your home-page.
* `page.html` — The layout for pages such as About, Contacts etc.
* `post.html` — The layout for your blog posts.

Whenever you switch to a new theme, the new theme might have a different set of layouts. For example, `Cayman` theme that I switched to only had `default.html` layout. That is why after a switch from `Minima` to `Cayman` there was a blank page instead of my blog's homepage (because `home.html` layout was missing).

To solve issue with missing layouts, in your blog's root directory create a `_layouts` folder, and inside that folder create 4 layout files. [Here](https://github.com/sergodeeva/sergodeeva.github.io/tree/master/_layouts) is an example of my layouts.

## Create includes
If you have some code snippets that you want to include in multiple layouts of your site, save the snippets as include files and insert them where required, by using the `include` tag. Using `includes` also improves the readability of your layouts' code.

Create `_includes` folder in the root directory of your site, and place includes files inside. For example, I created the following includes: `head.html`, `header.html`, `footer.html`. [Here](https://github.com/sergodeeva/sergodeeva.github.io/tree/master/_includes) is an example of my includes.

## Update _config.yml
Now you need to change theme in the `_config.yml` file:
{% highlight yaml %}
theme: jekyll-theme-cayman
{% endhighlight %}

Apart of that you might also need to add some additional settings to the `_config.yml` file - settings, that used by the new theme that you are switching to.
For example, for `Cayman` theme I needed to add `repository: sergodeeva/sergodeeva.github.io` to my `_config.yml`.

## Customize CSS styles
Once you done with updating `layouts`, `includes` and `_config.yml`, run a development server on your local machine:
{% highlight terminal %}
bundle exec jekyll serve
{% endhighlight %}

Now you should be able to browse your blog at `http://localhost:4000/`.

After you see the blog, you will probably be tempted to modify some colors, fonts or other CSS properties. By default Jekyll is looking up at theme gem's CSS files. However you can easily override them by creating a file `assets/css/style.scss` in the root directory of your site.

In the top of the file, add the following content:
{% highlight css %}
---
---

@import "{{ site.theme }}";
{% endhighlight %}

After that you can add any custom CSS (or Sass, including imports) immediately after the `@import` line.

Alternatively, you can copy `scss` files from the theme's GitHub repository and place them in the `_sass` folder of your site. [Here](https://github.com/sergodeeva/sergodeeva.github.io/tree/master/_sass) is an example of my `scss` files. You can beautify your site by modifying these `scss` files.

## Wrap up
This post was about getting you started with Jekyll themes, but there’s a lot that you can do if you are willing to spend some time to learn. Take a look at the [official Jekyll documentation](https://jekyllrb.com/docs/home/) for more information.

I hope you found this article useful. If you are trying to switch Jekyll theme using the steps mentioned above and get stuck anywhere, please ask a question in the comments below.

