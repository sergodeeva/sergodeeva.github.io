---
  layout: post
  title: "Falling in love with Python"
  date:   2018-05-31 10:23:00 +0800
  tags: python ilovecoding
  description: "My first attempt of wriring a web scraper in Python - code example"
---

I tried Python for the first time a couple of weeks ago. Can't say it was love from the first line of code, but after I replaced a standard PyCharm's Darcula theme with [Atom One Dark](https://plugins.jetbrains.com/plugin/8006-material-theme-ui), and after I tried to write a web scraper, I started enjoying quite a lot.

So I wrote a scraper that grabs the product data (product name, description, images, price) from one e-commerce website and saves it into CSV file. If the website owner could hear me, I would say sorry, and hope I did not create any issues for that website.

I think the code speaks better than any words, so here is my code (the `URL` is dummy just in case :) ):

{% highlight python %}
import requests
from bs4 import BeautifulSoup
import pandas as pd
from utils.transform import clean_description
import datetime
from time import sleep
from multiprocessing import Pool


URL = 'http://www.example.com'

def get_soup(url):
    headers = {
        'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.143 Safari/537.36'}
    timeout = 20
    response = requests.get(url, headers=headers, timeout=timeout)
    html = response.content
    return BeautifulSoup(html, "html.parser")


def get_category_urls(url):
    soup = get_soup(url)
    cat_urls = []
    categories = soup.find('ul', attrs={'id': 'leftnav'})
    for c in categories.findAll('a'):
        cat_urls.append(c['href'])
    return cat_urls


def get_product_urls(url):
    try:
        soup = get_soup(url)
        if soup.find('ul', attrs={"id": "products-grid-table"}):
            pages = soup.find('div', attrs={'class': 'page'}).text.split("of ", 1)[1]
            prod_urls = []

            for page in range(1, int(pages) + 1):
                soup = get_soup(url + "?page={}".format(page))
                product_urls_soup = soup.find('ul', attrs={'id': 'products-grid-table'})
                for row in product_urls_soup.findAll('a', attrs={'class': 'product-image'}):
                    prod_urls.append(row['href'])

            return prod_urls
    except Exception as ex:
        print("An error occurred while parsing a category page " + url + ": " + str(ex))
        return []


def get_product_details(url):
    try:
        soup = get_soup(url)
        sleep(1)
        if soup.find('div', attrs={"class": "product-shop"}).findAll('div')[2].find('span').text == "Available":
            prod_details = dict()
            product_id = soup.find('div', attrs={'class': 'model'}).find('span').text
            prod_details['title'] = soup.find('div', attrs={'class': 'product-name'}).find('h2').text.replace(
                product_id, "").lstrip()
            prod_details['description'] = clean_description(soup.find('div', attrs={'id': 'tab_product_details'}))
            prod_details['web_url'] = url
            prod_details['image_urls'] = ",".join(
                list(filter(None, map(lambda x: x['href'],
                                      soup.find('div', attrs={'class': 'product-img-box'}).findAll('a')))))
            prod_details['price'] = soup.find('span', attrs={"class": "price"}).text.split()[2].replace("S$", "")
            return prod_details
    except Exception as ex:
        print("An error occurred while parsing a product page " + url + ": " + str(ex))


start_time = datetime.datetime.now()
print("Started extracting data from " + URL + " at " + str(start_time))

category_urls = get_category_urls(URL)
print("Found {} category pages".format(len(category_urls)))

product_urls = [get_product_urls(category_url) for category_url in category_urls]
product_urls_flat = list(set([y for x in product_urls for y in x]))
print("Found {} product pages".format(len(product_urls_flat)))

with Pool(10) as p:
    products = p.map(get_product_details, product_urls_flat)
products = list(filter(None, products))

end_time = datetime.datetime.now()
print("Finished extracting data, the process took " + str(end_time - start_time))

products_df = pd.DataFrame(products)
products_df.drop_duplicates(subset=['product_id'], keep='first', inplace=True)
products_df.to_csv('product.csv', encoding='utf-8', index=False)
{% endhighlight %}

There are four functions:
- `get_soup` sends request to the given url and returns html response
- `get_category_urls` - returns a list of category urls from the homepage's menu
- `get_product_urls` - returns a list of product urls for the given category, takes into account pagination on the category page
- `get_product_details` - returns a product dictionary

I tried `multiprocessing` in order to speed up the scraping. Until now still can't understand what is the difference between `multiprocessing` and `threading`. Also, not really understand the syntax of it (I found it somewhere on the internet).

The most surprising thing for me was Pandas `drop_duplicates` method - without Pandas I might had to spend ages figuring out how to remove duplicates from the list of dictionaries (if a value of one specific key is duplicated).


I am thinking to improve the code further, e.g. in `get_soup` I should probably check if response status code is 200. If you have any suggestions of improvements, please write in the comments, it will be greatly appreciated. That's it for now, this was my first Python code and hope not the last.