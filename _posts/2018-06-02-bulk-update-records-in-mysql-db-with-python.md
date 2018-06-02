---
  layout: post
  title: "Bulk update records in MySQL DB with Python"
  date:   2018-06-02 12:19:00 +0800
  tags: python mysql
  description: "Using INSERT ON DUPLICATE KEY UPDATE"
---

Yesterday I spent a couple of hours trying to find the best way of updating multiple records in MySQL db using Python. Unfortunately could not find any really elegant way, as a result I ended up with using `'INSERT ON DUPLICATE KEY UPDATE'`.

Sharing my code here, hope it will save you time. If you know any better way of updating multiple rows in mysql db with python, let me know in the comments.

So, let's say I have a `students` table wich looks like this:

| id | name |
| ---- | ---- |
| 1 | cate blanchett |
| 2 | sia furler |
| 3 | priyanka chopra |

And let's say I want to capitalize each word in the `name` column (in reality I needed to apply more complex transformation, but will be talking about capitalizing for simplicity).

Here is the code that is doing a bulk update:

{% highlight python %}
from sqlalchemy import create_engine
import pandas as pd

sql_select = "SELECT id, name FROM students"

engine = create_engine('mysql+pymysql://root:123@127.0.0.1:3306/schooldb?charset=utf8')

students_df = pd.read_sql(sql_select, engine)
students_df['name'] = students_df['name'].str.title()
students_tp = str([tuple(prod) for prod in students_df.values]).strip('[]')

sql_update = "INSERT INTO students (id, name) VALUES " + students_tp + " ON DUPLICATE KEY UPDATE name=VALUES(name)"

engine.execute(sql_update)
{% endhighlight %}

Basically what I am doing here is:
- getting data from the DB table using `sqlalchemy`
- storing it into dataframe (to do so need to import `pandas`)
- applying the transformation I need (in this case I am capitalizing each word in the `name` column using `title()` function)
- converting the transformed dataframe to tuples (so that the data will look like `(1, 'Cate Blanchett'), (2, 'Sia Furler'), (3, 'Priyanka Chopra')`)
- updating the table using 'INSERT INTO' ... 'ON DUPLICATE KEY UPDATE' statement

This is how the database table looks like after I ran my script:

| id | name |
| ---- | ---- |
| 1 | Cate Blanchett |
| 2 | Sia Furler |
| 3 | Priyanka Chopra |

Easy. But I still feel there should be a better way...