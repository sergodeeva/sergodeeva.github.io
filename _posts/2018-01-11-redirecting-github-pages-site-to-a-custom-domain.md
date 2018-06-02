---
  layout: post
  title: "Redirecting GitHub Pages site to a custom domain"
  date:   2018-01-11 21:45:00 +0800
  tags: githubpages
  description: "Step by step instruction on how to setup your own domain with GitHub Pages"
  image: "/assets/post_images/2018-01-11-godaddy-dns.png"
---

A few years ago I had that bad habit to buy domain names. Could not stop myself from buying if I bumped into some good domain. This time, when building my current personal blog, I decided to use a standart GitHub Pages url (in my case sergodeeva.github.io). But last minute I changed my mind and bought "natalyakosenko.com".

The whole process of registering a domain and setting up a redirect from GitHub Pages to my new domain took me exactly 50 minutes:
<img src="{{ site.url }}/assets/post_images/2018-01-11-whatsapp.jpg" width="320" style="display:block"/>
(At 10:27 I was asking my brother's advise about domain name, and at 11:17 everything was done)

Below is a step by step instruction:

## Register domain
You can register a domain with your preferrable domain registrar. In my case I went to GoDaddy.com website, checked domain natalyakosenko.com - it was available. So I made payment using my credit card - domain name only, without purchasing any additional services. Domain name cost me about $US 40 for 5 years - not a bad price I think.

## Add domain to GitHub pages site
Now you need to decide whether you prefer using an apex domain (domain without "www", e.g. yourdomain.com), or "www" domain (e.g. www.yourdomain.com). In my case I chose www.natalyakosenko.com.
In your Github pages repository navigate to Settings section. Add your domain url (with or without "www") into "Custom domain" field, like this:
![Github Pages custom domain]({{ site.url }}/assets/post_images/2018-01-11-github-pages.png)

## Update domain DNS settings
The last step is updating domain's DNS settings with your DNS provider. Below are instructions for GoDaddy:
* Navigate to "DNS" section:
![Manage DNS]({{ site.url }}/assets/post_images/2018-01-11-godaddy-dns.png)

* If you are using domain with "www", update a value of `CNAME` record with your GitHub pages url.
* Create two `A` records that point your custom domain to the following IP addresses:
  * 192.30.252.153
  * 192.30.252.154

This is how final DNS settings should look like:
![Final DNS settings]({{ site.url }}/assets/post_images/2018-01-11-dns-settings.png)

That is it. If your GitHub Pages site isn't loading at your custom domain, you may have an error in your GitHub repository setup or your DNS configuration - double check all the settings.
