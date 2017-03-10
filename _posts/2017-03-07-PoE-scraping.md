---
layout: post
title: Path of Exile Scraping
---

To get the data for
my
[Path of Exile analysis]({{ site.baseurl }}{% post_url 2017-03-01-PoE-analysis %}) I
scraped all the leagues and ladders from the Path of Exile (PoE) official API
into an SQLite table. If you've never used SQLite before, it's easy and I'll
show you how I set it up so you can use it in your own projects. I'll also go
into detail about how to access the PoE API from Python.

## Setup

First we need to install SQLite and set up our python environment. The easiest
way to install SQLite on a Mac is to use [Homebrew](https://brew.sh/).

{% highlight bash %}
brew install sqlite
{% endhighlight %}

Then, assuming you have [conda](https://conda.io/docs/intro.html) installed
we'll create a new conda virtual environment for the project.

{% highlight bash %}
conda create -n poe_scrape python=2.7
{% endhighlight %}

And switch to the environment:

{% highlight bash %}
source activate poe_scrape
{% endhighlight %}

And lastly we'll need to grab and install the pathofexile github repository
which provides a wrapper for accessing the official api.

{% highlight bash %}
git clone https://github.com/willroberts/pathofexile
cd pathofexile
pip install -r requirements.txt
{% endhighlight %}

I had trouble installing the `gevent` dependency using pip so I ended up
installing it using conda:

{% highlight bash %}
conda install gevent==1.0.1
{% endhighlight %}

## Creating the SQLite table

Next we have to create a table to store the data we scrape. You can open a new
database and get to an interactive prompt by running sqlite from the command line.

{% highlight bash %}
sqlite3 poe_races.db
{% endhighlight %}

Now from the interactive SQLite prompt we can create two tables: one for the
leagues (with a unique id column and possibly non-unique description, startAt,
endAt, registerAt, url, rules, and event columns) and one for the ladders of
players that competed in those leagues.

{% highlight sql %}
CREATE TABLE leagues
             (id text PRIMARY KEY, 
              description text, 
              startAt text, 
              endAt text, 
              registerAt text, 
              url text, 
              rules text, 
              event integer);
{% endhighlight %}

{% highlight sql %}
CREATE TABLE ladders
             (id text, 
              account text, 
              twitch text, 
              challenges integer, 
              character text, 
              rank integer, 
              class text,
              experience integer,
              dead integer);
{% endhighlight %}

You can exit the prompt by typing `.quit` and hitting enter.

If you ever need to get rid of an entire table, such as when you're first testing your code, you can do so using the `DELETE` command.

{% highlight sql %}
DELETE * FROM ladders;
{% endhighlight %}

Later you might want to just delete a few records that match a certain condition (for example id='Accident').

{% highlight sql %}
DELETE FROM ladders
WHERE id='Accident';
{% endhighlight %}

## Scraping

Now that we have a tables in a database, we can start scraping to fill them up.
Leagues can be accessed from the API 50 at a time.

{% highlight python %}
import pathofexile.pathofexile.api as poe

league_list = poe.get_leagues(league_type='all',
                                      compact_info=0,
                                      league_limit=50,
                                      league_offset=0)
{% endhighlight %}

Now we can connect to the database and add our leagues. First we'll make a
connection and then grab a cursor that allows us to run SQL commands on the
database.

{% highlight python %}
import sqlite3

conn = sqlite3.connect('official_api_tools/poe_races.db')
c = conn.cursor()

def write_league_to_table(league, cursor, table):
    """ Writes a single league into an sqlite table
    
        :param league: Dictionary. Should contain 'id', 'description',
            'startAt', 'endAt', 'registerAt', 'url', 'rules', and 'event' keys.
        :param cursor: sqlite3.Cursor. A cursor connected to the database that
            contains the table to input leagues into.
        :param table: String. The table to insert leagues into.
        :return: Does not return.
    """ 
    league_tuple = (unicode(league['id']),
                    unicode(league['description']),
                    unicode(league['startAt']),
                    unicode(league['endAt']),
                    unicode(league['registerAt']),
                    unicode(league['url']),
                    unicode(league['rules']),
                    int(league['event']))
    cursor.execute('''  INSERT OR IGNORE INTO
                        {} VALUES (?,?,?,?,?,?,?,?)
                   '''.format(table), league_tuple)
                   
for league in league_list:
        write_league_to_table(league, c, 'leagues')
{% endhighlight %}

This queued our changes to the database, but the changes have not been made yet.
We need to "commit" the changes for them to take effect.

{% highlight python %}
conn.commit()
{% endhighlight %}
