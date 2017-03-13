---
layout: post
title: Path of Exile Analysis
---

# Path of Exile Leagues Analysis

[Path of Exile](http://www.pathofexile.com/) is an action role playing game made
by Grinding Gear Games (GGG) in a similar vein as Diablo. From its intricate,
unified skill tree to its purely barter economy, PoE has a number of unique
gameplay mechanics that cater to hard-core gamers. What really sets it apart,
though, are the races. In these events, which last anywhere from 12 minutes to
multiple months, players roll fresh characters, compete under a variety of
gameplay conditions, and generally try to run through the game faster than
everyone else.

If you have ever watched or played Path of Exile, you've probably had a number
of questions: Is there a preferred class? How consistent are the top racers? Are
certain races quantifiably more difficult than others (how _much_ worse is BLAMT
than Exiles Everywhere)? Puzzles solved, debates settled! I scraped the entire
Path of Exile database for the race results of 5,140,057 characters over 5,222
leagues to answer these very conundrums. You can read about how I did the
scraping [here]({{ site.baseurl }}{% post_url 2017-03-07-PoE-scraping%}).

## Basic info

First let's take a general look at the data. 

![League size distribution](/public/league_size_hist.png)

If we plot a histogram of the number of characters in each league, we see most
events have fewer than 2,000 characters. Path of Exile has a complete game outside
of races and there are a _lot_ of events in PoE so perhaps this isn't too
surprising.
A
[2015 press release](http://www.gamasutra.com/view/pressreleases/243342/Path_of_Exile_The_Awakening_Supporter_Packs_Achieve_RecordSales.php) put
the worldwide player base north of 10 million players, which means on the order
of 0.01% of players compete in any given event.

The bump in the histogram at 15,000 corresponds to the depth of the ladder
retrievable by the API (if a ladder had more than 15,000, I could only retrieve
the first 15,000 and it appeared to me to only have 15,000 characters). These
large events are mostly season-long leagues and the main game's perennial
Standard and Hardcore leagues. There is an outlier race that was only 3 days
long. The '3 Day Exiles Event HC (IC010)' was an in between season event
that
[awarded closed beta access](http://www.pathofexile.com/forum/view-thread/1293370) to
the next expansion for the top 200 players. Additionally Exiles Everywhere races
are especially unique, entertaining, and deadly (racers likely ended up having
multiple characters). In exiles everywhere, 20 rogue exiles (mini-bosses) spawn
per zone. You can check
out [ZiggyD](https://www.youtube.com/watch?v=t3TBU_Kwbes) in this event to
see just how brutal these races are.

Looking at how events have been distributed over time, we can see there are
generally 1000+ events a year. There is a noticeable up tick in events after
both the start of the open beta and the game's official release.

![Events over time](/public/poe_events_over_time.png)

## Classes over time

In any competitive game, the metagame is constantly in flux. Classes fade in and
out of popularity. After its release, the Scion quickly became the most played
class. Over time interest in the Scion has waned and in the past 12 months
players have started to favor racing Rangers.

![Class popularity over time](/public/poe_class_popularity_vs_time.png)

## Player consistency

A common debate in games that have randomly generated maps, monsters, and loot
drops is whether players win based on skill or luck. Can you actually win
consistently given the random number generator (RNG)? Yes, yes you can. Let's
look at one of PoE's most famous racers, Kripparrian. (The whiskers in the plot
correspond to 1.5x the range between the 25% and 75% quartiles)

![Kripp race consistency](/public/poe_kripp_consistency.png)

When he was active in the racing scene, Kripp dominated. He finished first 23%
(!) of the time, in the top ten in 69% of his races, and in the top 100 in 88%
of his races! To see just how amazing this is, let's look at the most recent
winner of the Medallion season: Steelmage.

![Steelmage race consistency](/public/poe_steelmage_consistency.png)

Over all their races, Steelmage placed first 3%, top ten 31%, and top 100 67% of
the time. While quite a ways from Kripp-level consistency, this is still pretty
impressive especially considering how long Steelmage has managed to keep up this
level of gameplay. Over roughly 3 years Steelmage has competed in 629 events
compared to Kripp's 259 in about 1 year. Since events aren't evenly distributed
but grouped into seasons, this means top racers are competing multiple times a
day, every day, during the month-long race leagues.

And for fun let's look at professional Youtuber and streamer ZiggyD who races
regularly, but not quite at the level of the previous two players.

![ZiggyD race consistency](/public/poe_ziggyd_consistency.png)

Over 133 events ZiggyD has placed first overall 0%, top ten 1.5%, and top 100
31% of the time. So yes there is some RNG in Path of Exile, but the very best
racers can consistently break the top 100 in the majority of their races, and
top ten more than a quarter of the time.

## Race difficulty

It seems clear that some races are more difficult than others: a Lethal mod race
where enemies do 50% extra normal damage and 50% normal damage as each element
should be more _lethal_ than a vanilla Burst race. But quantifying race
difficulty is not immediately straightforward. If we look at the experience
histogram for characters in an event, it always has roughly an exponential
decay.

![Race experience histogram](/public/poe_experience_example.png)

There's a pile up of characters at low levels who die early on and then a long
tail of high performers. I would claim that in the more difficult races there's
a larger disparity between the less-skilled majority who only manage to get a
small amount of experience points and the elite who accumulate many more
experience points. We can roughly quantify this using the skew of the
distribution [^Skew_statistic]. Skew measures how long and fat the experience
tail is.


[^Skew_statistic]:
    Specifically I used the adjusted Fisher-Pearson standardized moment
    coefficient G1 built into most statistical packages including pandas.

![Race difficulty](/public/poe_race_difficulty.png)

Let's digest this. The highest skew, most difficult leagues include Exiles
Everywhere, BLAMT, and Cutthroat. The "easier" leagues with less separation
between skill levels include vanilla Burst races and, generally, Endless Ledge.

The Exiles Everywhere race was mentioned earlier. In this race 20 mini-bosses
are spawned per area.

BLAMT is a masochistic race with a combination of modifiers: **B**lood magic,
**L**ethal, **A**ncestral, **M**ulti-projectile, **T**urbo. This means
characters use health instead of mana for skills, monsters do 50% extra normal
damage and 50% normal damage as each element, there are many more enemey buff
totems, enemies fire four additional projectiles, _and_ monsters move, cast, and
attack 60% faster. Lethal and Turbo races have just their modifiers,
respectively.

Cutthroat centers around killing other players for loot and experience. Not
surprisingly most characters get killed at low levels while some snowball by
coming out on top early.

In Headhunter races players acquire bonuses after killing rare monsters which
are much more plentiful in this league, but still just as deadly.

Endless Ledge is a straight run in a fixed looping area (Ledge) with
increasingly difficult monsters after each loop.

In contrast Burst races have no modifiers and, like the other races, the
objective is to get as much experience as possible in the time limit. It makes
sense that there is a lot less skew in experience for a typical 12 minute Burst
race.

It looks like, on average, Exiles Everywhere races do just a little bit better
job than BLAMT races of separating the pros from the newbs. Surprised?

## Where to from here?

To see the code behind this post (and some more analysis that didn't make the
cut!) checkout
the [github repository](https://github.com/stevejbrown/PoE-analysis). I
encourage you to play with the data yourself. There are answers to so many
questions buried in this data set: how has the winning class changed over time?
What is the distribution of race lengths? Are some players better at certain
races than others? Which races are going in and out of style? How did the
Russian Garena Realm differ from the main GGG realm? Which races are most
popular? Happy digging!
