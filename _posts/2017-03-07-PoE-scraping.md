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
which provides a wrapper for accessing the official api. It currently doesn't
have a setup.py to import modules from it, so it needs to be cloned in the
project directory where we'll do the scraping. We'll still need to install its
requirements of course.

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

Next we can connect to the database and add our leagues. First we'll make a
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

Note that I used INSERT OR IGNORE. This attempts to insert a league into the
database, but if it already exists in the database (has the same unique/primary
'id' key) then it silently does nothing instead of throwing an error. Running
this code queued our changes to the database, but the changes have not been made
yet. We need to "commit" the changes for them to take effect.

{% highlight python %}
conn.commit()
{% endhighlight %}

Similarly, we can scrape the ladders. Here's an example for the league '3 Day
Exiles Event HC (IC010)'.

{% highlight python %}
import pathofexile.pathofexile.ladder as poe_ladder

league_id = '3 Day Exiles Event HC (IC010)'
ladder = poe_ladder.retrieve(league_id)

for entry in ladder:
    if 'twitch' in entry['account']:
        twitch = entry['account']['twitch']['name']
    else:
        twitch = 'NULL'
    if 'experience' in entry['character']:
        experience = int(entry['character']['experience'])
    else:
        experience = None
    ladder_tuple = (unicode(league_id),
                    unicode(entry['account']['name']),
                    unicode(twitch),
                    int(entry['account']['challenges']['total']),
                    unicode(entry['character']['name']),
                    int(entry['rank']),
                    unicode(entry['character']['class']),
                    experience,
                    int(entry['dead']))
    cursor.execute(''' INSERT OR IGNORE INTO {}
                        VALUES (?,?,?,?,?,?,?,?, ?)
                    '''.format(ladder_table), ladder_tuple)
                    
conn.commit()
{% endhighlight %}

## Cleaning up

I often had to restart the scraper due to hitting a rate limit or other error
and, because of this, I would accidentally scrape the same entry more than once.
Since the ladders table didn't have any unique keys I ended up having many
duplicates. We'd like to remove these duplicate entries. The idea for removing
them is simple: if multiple rows have the same value for every column, then get
rid of each of these duplicate rows except the first. The trick is to GROUP BY
all the columns (which groups identical rows together) and then DELETE all
except the lowest numbered rowid in each group.

{% highlight sql %}
DELETE FROM ladders
          WHERE rowid NOT IN(
             SELECT  MIN(rowid)
             FROM    ladders
             GROUP BY id, account, twitch, challenges, character, 
                      rank, class, experience, dead
             );
{% endhighlight %}

Now the database is ready to be analyzed!

## Final notes

I ran into a few snags when scraping the PoE data and needed to patch the
pathofexile library. The changes have already been merged into the library, so
you shouldn't have to worry about them, but they're good to keep in mind for
your own scraping projects. 

The pathofexile library is written in Python 2 which
doesn't treat strings as unicode out of the box. Since most of the Russian PoE
leagues use Cyrillic characters (for example Флешбэк одна жизнь (IC002)) I had
to explicitly use unicode strings (`u'Флешбэк одна жизнь (IC002)'` or
`unicode('Флешбэк одна жизнь (IC002)')`). Also some leagues had a "/" in them
(e.g. 'OneWeekHCRampage/Beyond'). This doesn't seem like a big deal but
pathofexile is setup to create a cache for when you are repeatedly polling
ladders and uses the league name to name the file. This turns into an error as
it creates a directory instead of a file. I got around this by just substituting
"_" for "/" (`'string'.replace('/','_'`).

You can find the full code I used for the scraping
on [github](https://github.com/stevejbrown/PoE-analysis). And make sure to read
the [analysis article]({{ site.baseurl }}{% post_url 2017-03-01-PoE-analysis %})
to see all the insights buried in this data!
