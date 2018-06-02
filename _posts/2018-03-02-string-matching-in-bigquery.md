---
  layout: post
  title: "String matching in BigQuery"
  date:   2018-03-02 21:15:00 +0800
  tags: bigquery
  description: "Matching two lists of geographical locations using BigQuery"
  image: "/assets/post_images/2018-03-02-bq.jpg"
---

I've got some interest in BigQuery recently. I've been using BQ for a few years, but did not enjoy it much: did not like the BigQuery UI and did not like the fact that data needs to be duplicated in a regular database (e.g. MySQL, for production usage) and in BigQuery (for analysis purposes).

However a few weeks ago I needed to compare two lists of geographical locations in BigQuery and find matches. The first list contained about 100k geographical locations, and the second one - about 30k locations. Both lists looked like this:

| id  | name     | state_code | country_code |
| --- | -------- | ---------- | ------------ |
| 1   | Moscow   | null       | Russia       |
| 2   | Budapest | null       | Hungary      |
| 3   | Calgary  | AB         | Canada       |

## Step 1: getting matches using SELECT

So I wrote a simple query to get locations that are presented in both lists:
{% highlight sql %}
SELECT l1.id, l2.id FROM locations1 l1
JOIN locations2 l2 ON l1.name = l2.name
WHERE l1.country_code = l2.country_code
(AND l1.state_code = l2.state_code OR l1.state_code IS NULL);
{% endhighlight %}

However how to deal with the cases when locations are named slightly differently? E.g. in `locations1` I have "St Petersburg" and in `locations2` I have "Saint Petersburg" or "St. Petersburg". Very quickly I found major cases that causing differences:

* Thai islands: "Koh Samet" vs "Ko Samet"
* Missing spaces: "Chiang Mai" vs "Chiangmai",
* Missing dashes: "Ulan-Ude" vs "Ulan Ude"
* Strokes: "Noril'sk" vs "Norilsk"

## Step 2: removing special characters

I created a temporary function in BigQuery to normalise the string: convert it to lowercase, remove special characters, replace "saint" with "st" and "koh" with "ko".

The function is:
{% highlight sql %}
CREATE TEMP FUNCTION rmSpecCharacters(name_en STRING) AS
((
  SELECT REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(string,"saint ","st "), " saint", " st"),"sankt ","st ")," sankt"," st"),"koh ", "ko ")," ","") FROM (
    SELECT REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(LOWER(name_en),"´",""),"'",""),"ʼ",""),".",""),"-"," ") as string
)
));
{% endhighlight %}

As you might have noticed in this query I do not handle cases like "Calangute Beach" and "Calangute", because sometimes it can be two different locations (e.g. "Miami" and "Miami Beach" are two different cities).

## Step 3: replacing accented characters with latin characters

In oppose to MySQL (I am using 5.6.34), in Big Query "á" != "a". So this query returns `true` in Sequel Pro, but it returns `false` in BQ:
{% highlight sql %}
SELECT "á" = "a"
{% endhighlight %}

As a result BigQuery treats "Düsseldorf" and "Dusseldorf" as two different strings. So I created one more temp function to replace accented characters with latin:
{% highlight sql %}
CREATE TEMP FUNCTION accent2latin(name_en STRING) AS
((
  WITH lookups AS (
    SELECT 
    "ç,æ,œ,á,é,í,ó,ú,à,è,ì,ò,ù,ä,ë,ï,ö,ü,ÿ,â,ê,î,ô,û,å,ø,Ø,Å,Á,À,Â,Ä,È,É,Ê,Ë,Í,Î,Ï,Ì,Ò,Ó,Ô,Ö,Ú,Ù,Û,Ü,Ÿ,Ç,Æ,Œ,ñ,š,ý,ž,ě,õ,ň,č,ř,ť,ł,ő,ń,ź,ś,ć,ę,ã,ā,ū,ī,ð,ţ,ş,ï,ė,đ,ō,ġ,ż,ı,ç,ğ" AS accents,
    "c,ae,oe,a,e,i,o,u,a,e,i,o,u,a,e,i,o,u,y,a,e,i,o,u,a,o,O,A,A,A,A,A,E,E,E,E,I,I,I,I,O,O,O,O,U,U,U,U,Y,C,AE,OE,n,s,y,z,e,o,n,c,r,t,l,o,n,z,s,c,e,a,a,u,i,d,t,s,i,e,d,o,g,z,i,c,g" AS latins
  ),
  pairs AS (
    SELECT accent, latin FROM lookups, 
      UNNEST(SPLIT(accents)) AS accent WITH OFFSET AS p1, 
      UNNEST(SPLIT(latins)) AS latin WITH OFFSET AS p2
    WHERE p1 = p2
  )
  SELECT STRING_AGG(IFNULL(latin, char), '')
  FROM UNNEST(SPLIT(name_en, '')) char
  LEFT JOIN pairs
  ON char = accent
));
{% endhighlight %}
(Original code was copied from [here](https://stackoverflow.com/questions/43145902/bigquery-convert-accented-characters-to-their-plain-ascii-equivalents/43148949), however I enhanced it by adding Turkish, Czech and some other accented characters).

## Step 4: calculating string similarity using Levenshtein algorithm
After I "normalised" location names by removing special characters, replacing dashes and parts of the string and replacing accented characters, I was able to match two lists quite accurately. However it still was not perfect as I was not able to match similar names such as "Naberezhnye Chelny" and "Naberezhnie Chelny" for example.

To measure string similarity I decided to use [Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance) algorithm.
So I created a temporary BigQuery function that calculates similarity of two given strings:

{% highlight sql %}
CREATE TEMPORARY FUNCTION levenshtein(a STRING, b STRING)
RETURNS FLOAT64
LANGUAGE js AS """
  if(a == null) return 0;
  if(b == null) return 0;
  if(a.length == 0) return b.length; 
  if(b.length == 0) return a.length; 

  var matrix = [];

  var i;
  for(i = 0; i <= b.length; i++){
    matrix[i] = [i];
  }

  var j;
  for(j = 0; j <= a.length; j++){
    matrix[0][j] = j;
  }

  for(i = 1; i <= b.length; i++){
    for(j = 1; j <= a.length; j++){
      if(b.charAt(i-1) == a.charAt(j-1)){
        matrix[i][j] = matrix[i-1][j-1];
      } else {
        matrix[i][j] = Math.min(matrix[i-1][j-1] + 1,
                                Math.min(matrix[i][j-1] + 1,
                                         matrix[i-1][j] + 1));
      }
    }
  }
  
  var strlen = Math.max(a.length, b.length);

  return 1.0 - (matrix[b.length][a.length] / strlen);
""";
{% endhighlight %}

Similarity is a number between 0 and 1. Per my observations, if two location names have similarity 0.8 and above, they are most likely the same. However it is not always the case. For example, similarity of "Bangalore" and "Bengaluru" is 0.66 only.

## What's next
Using BigQuery temporary functions, I was able to match two lists of geo locations quite fast and accurately. If there was no BiqQuery, I could write some code in Ruby perhaps, however I might had to spend more time.

I am currently thinking on how to reduce false-positive matches of Levenshtein algo. For example, "South Palmetto Point" and "North Palmetto Point" have similarity score 0.89, however if Levenshtein algorithm knew English, it would know that "South" and "North" are two opposite things :) Or another example, "Pano Akourdalia" and "Kato Akourdalia" have 0.86 similarity score, but "Pano" and "Kato" in Greek language mean "Over" and "Below" accordingly.

I'm wondering if I could use maching learning for that?..