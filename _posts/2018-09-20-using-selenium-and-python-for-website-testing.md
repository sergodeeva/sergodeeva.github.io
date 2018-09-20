---
  layout: post
  title: "Using Selenium WebDriver and Python for site testing"
  date:   2018-09-20 19:59:00 +0800
  tags: python
  description: "How to use Selenium & Python for frontend testing (with code examples)"
  image: "/assets/post_images/2018-09-20-software-bugs.jpg"
---

Have you ever had a situation when a website you are working on got some issues, and you knew about it not from your monitoring dashboards/alerts but from the end users or from your manager? For example, autosuggest stopped working, or crucial for SEO metatags misteriously disappeared from some pages, or a button suddenly became unclickable.

<img src="{{ site.url }}/assets/post_images/2018-09-20-software-bugs.jpg" style="display:block"/>

It happened to me, and I can tell there is no pleasure when someone finds your/your team mates bugs before you found them. Luckily, it happens to me not that often any more, as recently my colleague Nik enlightened me about website testing with Python and Selenium.

How is it different from other kinds of testing? Basically the idea is to write a humanlike bot that will visit your website and do some simple actions: click buttons, type queries in autosuggest field, verify presence of webpage elements, verify meta tags etc. If something is not as expected, the bot will notify you.

But enough words, let me show you some code. I will be testing my own blog natalyakosenko.com:
- verify that the homepage has right title,
- verify that the homepage has one and only one h1 header,
- verify that all the post links on the homepage are not broken.

To be honest, you do not need to use Selenium for such simple cases. Can use `requests` instead. But I am using these examples for simplicity, to explain the idea. So here is the code:

{% highlight python %}
import requests
import sys
import unittest
from selenium import webdriver


class HomepageTest(unittest.TestCase):
    def setUp(self):
        self.driver = webdriver.Chrome()
        self.url = 'http://www.natalyakosenko.com/'

    def test_homepage_title(self):
        driver = self.driver
        driver.get(self.url)
        self.assertIn('Kosenko', driver.title)

    def test_homepage_h1(self):
        driver = self.driver
        driver.get(self.url)
        h1 = driver.find_elements_by_tag_name('h1')
        self.assertEqual(1, len(h1))

    def test_broken_links(self):
        driver = self.driver
        driver.get(self.url)
        links = driver.find_elements_by_class_name('post-link')
        for link in links:
            try:
                response = requests.head(link.get_attribute('href'))
                self.assertTrue(response.status_code in (200, 201))
            except Exception as ex:
                self.fail(f"Broken link detected {link.get_attribute('href')}: {ex}")

    def tearDown(self):
        self.driver.close()


if __name__ == "__main__":
    test_suite = unittest.TestSuite()
    test_suite.addTest(HomepageTest())
    unittest.main(argv=[sys.argv[0]], warnings='ignore')

{% endhighlight %}

In this example, I have one test suite, `HomepageTest` class. In reality you might have many test suites, each test suite contains tests that are grouped together according to pages/features they test.

Inside `HomepageTest` there are `setUp` and `tearDown` - standard functions that contain prerequisite steps and clean-up steps for the test suite. You might be wondering, why these functions are named in camelCase instead of traditional pythonic snake_case? Well, I was wondering too, and this is what I found on [python.org](https://docs.python.org/3.1/library/unittest.html):

> The Python unit testing framework, sometimes referred to as “PyUnit,” is a Python language version of JUnit, by Kent Beck and Erich Gamma. JUnit is, in turn, a Java version of Kent’s Smalltalk testing framework. Each is the de facto standard unit testing framework for its respective language.

In this line, the instance of Chrome driver is created:
{% highlight python %}
self.driver = webdriver.Chrome()
{% endhighlight %}
I used Chrome driver, but other browsers are also supported (e.g. Firefox, IE etc). If you get `selenium.common.exceptions.WebDriverException: Message: 'chromedriver' executable needs to be in PATH` error, you might want to use a solution suggested [here](https://stackoverflow.com/questions/29858752/error-message-chromedriver-executable-needs-to-be-available-in-the-path).

My test suite contains 3 simple tests:

- `test_homepage_title()` - it asserts that the site homepage contains my name in the title:
{% highlight python %}
self.assertIn('Kosenko', driver.title)
{% endhighlight %}

- `test_homepage_h1` - this test finds all `h1` headers on the page and asserts that there is only one header h1 is found:
{% highlight python %}
h1 = driver.find_elements_by_tag_name('h1')
self.assertEqual(1, len(h1))
{% endhighlight %}

- `test_broken_links` - here I am finding all the post links on the homepage (links that have CSS class 'post-link'), then looping through them and asserting that response status is either 200 or 201:
{% highlight python %}
links = driver.find_elements_by_class_name('post-link')
for link in links:
    try:
        response = requests.head(link.get_attribute('href'))
        self.assertTrue(response.status_code in (200, 201))
    except Exception as ex:
        self.fail(f"Broken link detected {link.get_attribute('href')}: {ex}")
{% endhighlight %}

That is it.

The main downside of Selenium tests to me is that writing of more complex scenarios is quite time-consuming, especially for those who have different desktop/mobile sites - you will need to write everything twice. Also, it can't fully protect your website from bugs and should be used together with other kinds of testing. Nevertheless, despite of some downsides it really helps me to sleep better at nights :)

P.S. Btw, while writing this post and testing my code, I found a broken link on my site's homepage.
