---
  layout: post
  title: "Writing Context Managers in Python"
  date:   2018-10-05 16:25:00 +0800
  tags: python
  description: "How to make python code more elegant using 'with' statement"
  image: "/assets/post_images/2018-10-05-confetka.jpg"
---

Today was a happy day: for the first time in my life I consciously used a python context manager. When I [started coding with Python](http://www.natalyakosenko.com/2018-05-31-falling-in-love-with-python) in May this year, I struggled to get the idea of context managers. Then I read a book ["Python Tricks" by Dan Bader](https://realpython.com/products/python-tricks-book/), which brought me out of darkness. Since then I was waiting for some real-life situation where I can apply my new knowledge. And today it happened!

So today I was writing a web scraper that supposed to get some information from the site X. Before scraping I needed to login: fill in a form and submit it, that is why I decided to use Selenium Webdriver. My first impulse was to write something like this:

{% highlight python %}
driver = webdriver.Chrome()
try:
    driver.get('http://www.natalyakosenko.com/')

    email = driver.find_element_by_name('email')
    password = driver.find_element_by_name('password')

    email.send_keys('my_email')
    password.send_keys('my_password')
    driver.find_element_by_name('submit').click()
finally:
    driver.close()
{% endhighlight %}

Basically, this code creates an instance of Chrome webdriver, then tries to do some stuff (in this case fills in email/password and presses Submit button), and finally `close` method is called to close a browser.

Then I realised that I will need to use Selenium Webdirver in other parts of my code, and I did not want to instantiate driver and write try/catch again and again. And this is where I thought: what a great opportunity to try a `with` statement in action!

The previous code written using `with` statement:
{% highlight python %}
class SeleniumDriver:
    def __enter__(self):
        self.driver = webdriver.Chrome()
        return self.driver

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.driver:
            self.driver.close()


with SeleniumDriver() as driver:
    driver.get('http://www.natalyakosenko.com/')

    email = driver.find_element_by_name('email')
    password = driver.find_element_by_name('password')

    email.send_keys('my_email')
    password.send_keys('my_password')
    driver.find_element_by_name('submit').click()
{% endhighlight %}

What is the benefit, you might think, instead of 12 lines of code got 19. But I do not need to define SeleniumDriver class every time. Whenever I need to use Selenium Webdriver in other parts of the code, I can simply write this:

{% highlight python %}
with SeleniumDriver() as driver:
    driver.get('http://www.natalyakosenko.com/')

    email = driver.find_element_by_name('email')
    password = driver.find_element_by_name('password')

    email.send_keys('my_email')
    password.send_keys('my_password')
    driver.find_element_by_name('submit').click()
{% endhighlight %}

Isn't it beautiful?
