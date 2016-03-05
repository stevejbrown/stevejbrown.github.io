---
layout: post
title: Journal article recommender
---
    
## Introduction

I like to keep up on the the current academic literature, but there are so many
papers published each week it’s hard to even find the relevant papers to read. I
created a tool so a computer can do the searching for me.

Previously there were two main ways to stay up to date: alerts or RSS feeds. RSS
feeds give you all the recent articles from a particular publisher, but you have
to sort through them manually to find relevant articles. Alerts are more
specific and have the advantage that they automatically notify you when a paper
either has a particular keyword or citation. Unfortunately with a small set of
keywords you are likely to create an alert that is much too broad (e.g.
'graphene') or too specific ('Dyakonov waves'). 

What would really be nice is an alert system that has a more holistic view of
what you're interested in. Something that knows the papers you have been
interested in in the past so it can predict new articles you'd like to read.
Before I started this project I was pretty excited about the ReadCube citation
manager. One of its promised features was personalized recommendations based on
your library of read papers. Perfect! ...Until I found out that the feature had
been "coming soon" for over a year.[^ReadCube_recommendation]

I decided to implement such a personalized recommender myself. It would rank new
articles based on how similar they are to articles I have already read.

[^ReadCube_recommendation]:[ReadCube](https://www.readcube.com/#feature-recommendations) has finally released this feature, but I have yet to try it out.

<!--- Need to describe the recommender framework first --->

## Model

As a first pass I decided to use **bag of words** to represent articles
and **cosine similarity** to rank the article "word bags".

Bag of words takes a text corpus and turns it into a vector of numbers which is 
much easier to compare than the raw text. In a bag of words vector, each element
of the vector corresponds to a word and the value of the element corresponds to
the number of times that the word appears in the corpus.

![bag_of_words](/public/bag_of_words.png "Bag of words")

Since only the number of times each word appears is counted, but not the
position of the word nor its relationship to other words, you basically
represent a text as just a grab bag of words. Because of this it's simple, fast,
easy to implement, and easy to understand. But it can be overly simple since it
ignores the structure that the words are in and multiple variations of the same
word ('aluminum' and 'aluminium') are treated as different words. 

In practice vectors need to have a finite number of dimensions so we choose
to only have the n most common words in the training corpus make up our
vectors (in my case n = 1000). Importantly only these words will be counted in
new articles so we have a fixed basis.

Now I'd like to rank new articles based on their similarity to my library. By
comparing the bag of words vector of a new article to the average vector of my
library, we can see how similar new articles are to the prototypical article I
would read. I use cosine similarity to make this quantitative. Cosine
similarity compares how similar the two vectors are by finding the cosine of the
angle between them (\\(\cos{\theta} = \frac{U \cdot V}{\|U\| \|V\|}\\)). If two
vectors are pointing in the same direction, the angle between them will be small
and the cosine of that angle will be near 1. With the single number score from
cosine similarity I can directly rank how close new articles are to my library.


## Implementation

To implement the article recommender I chose to go with a modular design where
loading the library to train against, training and scoring, and loading new
articles to rank would each be separate classes. This allows me to add new
components in the future, such as getting new articles from Google Scholar or
loading a BibTeX library, without having to rewrite the core scoring algorithms.
I've simplified some of the code for readability, but the general flow is as follows:

<ol>
    <li> <b>Get training articles:</b> First get article titles and abstracts
    from my library of read articles. I use the open source reference
    manager <a href="https://www.zotero.org/">Zotero</a> to store my
    articles. Exporting this as a csv file is the quickest way to get started.</li>

{% highlight python %}
import pandas as pd

full_library = pd.read_csv(library_csv)
library = full_library[['Title','Abstract Note']]
library.rename(columns={'Abstract Note': 'Abstract'},
               inplace=True)
{% endhighlight %}


    <li> <b>Preprocess:</b> Strip anything besides letters and spacing using the
    <a href="https://docs.python.org/2/library/re.html">regular expressions
    library</a>, make all text lowercase, and remove common stop words (‘the’,
    ‘and’, ‘a’, etc.) using the <a href="http://www.nltk.org/">Natural Language
    Toolkit</a>.

{% highlight python %}
import re
from nltk.corpus import stopwords

def row_to_words(row):
    ''' Converts a raw library entry to a string of words
        Input: A single row from the library
        Output: A single processed string
    '''
    # Merge columns
    text = row.fillna("")
    text = "".join(text)

    # Remove non-letters
    letters_only = re.sub("[^a-zA-z]"," ", text)

    # Convert to lower case, split into individual words
    words = letters_only.lower().split()

    # load stop words (searching a set to check if it includes
    # an element is much faster than searching a list) 
    stops = set(stopwords.words("english"))

    # Remove stop words
    meaningful_words = [w for w in words if not w in stops]

    # Join the words back into one string separated by a space
    # and return the result
    return(" ".join(meaningful_words))
{% endhighlight %}
    </li>

    <li><b>Vectorize:</b> Create a bag of words vectorizer based on my library
    using scikit-learn’s <a
    href="http://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.CountVectorizer.html">CountVectorizer</a>
    and find the average word vector to compare new articles against.

{% highlight python %}
from sklearn.feature_extraction.text import CountVectorizer

count_vectorizer = CountVectorizer(analyzer = "word",
                                   tokenizer = None,
                                   preprocessor = None,
                                   stop_words = None,
                                   max_features = 1000)
                                        
def train_vectorizer(count_vectorizer, cleaned_library)
    features = count_vectorizer.fit_transform(cleaned_library)
    features = features.toarray()
    ave_vec = features.sum(axis=0)
    ave_vec = ave_vec/np.linalg.norm(ave_vec) # Normalize
    return ave_vec
{% endhighlight %}</li>

    <li><b>Get new articles:</b> Pull journal RSS feeds with recently published
    articles from the feed aggregator <a href="https://feedly.com">Feedly</a>.
    The <a href="https://github.com/WarmongeR1/python-feedly">python-feedly</a>
    library provides wrappers to streamline common operations.
    
{% highlight python %}
from feedly import client as feedly

with open('inputs/feedly_client_token.txt', 'r') as f:
    FEEDLY_CLIENT_TOKEN = f.readline()
    
client = feedly.FeedlyClient(token = FEEDLY_CLIENT_TOKEN, 
                             sandbox = False)
streams = {}

def load_stream(feed, num_articles=500):
    ''' Loads unread articles in a feedly stream. Optional  
        parameter num_articles sets how many articles to load.
    '''
    stream = client.get_feed_content(
                 access_token=FEEDLY_CLIENT_TOKEN,
                 streamId=feed[u'id'], unreadOnly='true',
                 count=num_articles)
    return stream
{% endhighlight %}</li>

    <li><b>Preprocess new articles:</b> Extract title and abstracts from RSS
    HTML using <a
    href="http://www.crummy.com/software/BeautifulSoup/">BeautifulSoup</a>.
    Clean and vectorize the text the same way as the training samples (function omitted).
    
{% highlight python %}
from bs4 import BeautifulSoup

def get_content(stream):
    ''' Get the title and abstract of items in an rss stream '''
    content_list = []
    for item in stream[u'items']:
        title = item[u'title']
        title_text = remove_html(title)
        try:
            abstract = item[u'content'][u'content']
            abstract_text = remove_html(abstract)
        # The article may not contain a content field
        except KeyError:
            article_text = ""
        # The clean function is nearly identical to row_to_words
        article_text = (clean(title_text) + " "
                        + clean(abstract_text))
        content_list.append(article_text)
    return content_list
    
def remove_html(rss_field):
    ''' Removes the HTML tags from specific rss fields '''
    # Get the longest paragraph tag which usually removes the 
    # journal name, doi, or author list 
    soup = BeautifulSoup(rss_field)
    longest_p = max(soup.find_all("p"), 
                    key=lambda tag: len(unicode(tag)))

    # Remove HTML
    field_text = longest_p.get_text()
    return field_text
{% endhighlight %}
    
    </li>

    <li><b>Rank:</b> Compare the new articles to the average training article
    vector using cosine similarity.
    
{% highlight python %}

def score_article(ave_vec, article, vectorizer):
    '''
    Scores an article string by cosine similarity to average 
    library vector. It  adds an additional 1 in the 
    normalizing denominator to prevent divide by zero for
    empty article strings.

    Returns a numpy array with the article score as a 
    single element.
    '''

    article_vec = vectorizer.transform([article])
    article_vec = article_vec.toarray()
    score = (np.dot(article_vec, ave_vec)/
             (np.linalg.norm(article_vec)+1))
    return score
    {% endhighlight %}</li>
</ol>

## Outcome

This program quickly ranks a week’s worth (500-600 papers) of new papers where
about 7 of the top 10 ranked papers are relevant and nearly all papers relevant
to my research in are in the top 30. This ends up saving roughly 30-45 minutes
of manual sifting per week and allows me to look for articles in more niche
journals.

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Title</th>
      <th>Url</th>
      <th>Score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Coupling between diffusion and orientation of pentacene molecules ...</td>
      <td>http://feeds.nature.com/~r/nmat/rss/aop/~3/5WDMrZ2nT_4/nmat4575</td>
      <td>0.313582</td>
    </tr>
    <tr>
      <td>Chiral atomically thin films</td>
      <td>http://feeds.nature.com/~r/nnano/rss/aop/~3/QsHzpeSSRpg/nnano.2016.3</td>
      <td>0.294491</td>
    </tr>
    <tr>
      <td>Controlling spin relaxation with a cavity</td>
      <td>http://feeds.nature.com/~r/nature/rss/current/~3/ciM-agFY1BI/natur...</td>
      <td>0.294426</td>
    </tr>
    <tr>
      <td>Direct measurement of exciton valley coherence in monolayer WSe2</td>
      <td>http://feeds.nature.com/~r/nphys/rss/aop/~3/y1wbRV0DAl8/nphys3674</td>
      <td>0.274917</td>
    </tr>
    <tr>
      <td>Electrostatic catalysis of a Diels–Alder reaction</td>
      <td>http://feeds.nature.com/~r/nature/rss/current/~3/9EIXjtxOevw/natur...</td>
      <td>0.247099</td>
    </tr>
    <tr>
      <td>Multi-wave coherent control of a solid-state single emitter</td>
      <td>http://feeds.nature.com/~r/nphoton/rss/current/~3/fM48f_mmHSc/npho...</td>
      <td>0.247000</td>
    </tr>
    <tr>
      <td>Electro-optic sampling of near-infrared waveforms</td>
      <td>http://feeds.nature.com/~r/nphoton/rss/current/~3/ih2CR8lkn54/npho...</td>
      <td>0.240009</td>
    </tr>
    <tr>
      <td>Self-homodyne measurement of a dynamic Mollow triplet in the solid...</td>
      <td>http://feeds.nature.com/~r/nphoton/rss/current/~3/FL8iMhrGP-g/npho...</td>
      <td>0.234039</td>
    </tr>
    <tr>
      <td>Chiral magnetic effect in ZrTe5</td>
      <td>http://feeds.nature.com/~r/nphys/rss/aop/~3/9TMnsh32Hi8/nphys3648</td>
      <td>0.230376</td>
    </tr>
    <tr>
      <td>Condensation on slippery asymmetric bumps</td>
      <td>http://feeds.nature.com/~r/nature/rss/current/~3/Ukr2itGNCX0/natur...</td>
      <td>0.229368</td>
    </tr>
    <tr>
      <td>Realization of a tunable artificial atom at a supercritically char...</td>
      <td>http://feeds.nature.com/~r/nphys/rss/aop/~3/kBJieWtqays/nphys3665</td>
      <td>0.228723</td>
    </tr>
    <tr>
      <td>Experimental realization of two-dimensional synthetic spin–orbit c...</td>
      <td>http://feeds.nature.com/~r/nphys/rss/aop/~3/e9bpO5vT08Y/nphys3672</td>
      <td>0.222773</td>
    </tr>
    <tr>
      <td>Collective magnetic response of CeO2 nanoparticles</td>
      <td>http://feeds.nature.com/~r/nphys/rss/aop/~3/Hob8ZdJ5bh4/nphys3676</td>
      <td>0.221313</td>
    </tr>
    <tr>
      <td>Observation of room-temperature magnetic skyrmions and their curre...</td>
      <td>http://feeds.nature.com/~r/nmat/rss/aop/~3/cpzy9V09J0M/nmat4593</td>
      <td>0.210910</td>
    </tr>
    <tr>
      <td>Coherent control with a short-wavelength free-electron laser</td>
      <td>http://feeds.nature.com/~r/nphoton/rss/current/~3/H4xCuMGuyZM/npho...</td>
      <td>0.206225</td>
    </tr>
  </tbody>
</table>
