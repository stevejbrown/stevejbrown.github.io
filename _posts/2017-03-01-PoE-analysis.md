---
layout: post
title: Path of Exile Analysis
---

# Path of Exile Leagues Analysis

Path of Exile is an action role playing game in a similar vein as Diablo. From
its intricate, unified skill tree to its purely barter economy, PoE has a number
of unique gameplay mechanics that cater to hard-core gamers. What really sets it
apart, though, are the races. In these events which last anywhere from 12
minutes to multiple months, players roll fresh characters, compete under a
variety of gameplay conditions, and generally try to run through the game faster
than everyone else.

If you have ever watched or played Path of Exile, you probably had a number of
questions: Is there a preferred class? How consistent are the top racers? Are
certain races quantifiably more difficult than others (how _much_ worse is BLAMT
than exiles everywhere)? Puzzles solved, debates settled! I scraped the
entire Path of Exile database for the race results of 5140057 characters over
5222 leagues to answer these very queries [perplexities, conundrums].

## Basic info

First let's take a general look at the data. 

![League size distribution](/public/league_size_hist.png)

If we plot a histogram of league size, we see most events have less than 2000
players. Path of Exile has a complete game outside of races and there are a
_lot_ of events in PoE so perhaps this isn't too surprising.
A
[2015 press release](http://www.gamasutra.com/view/pressreleases/243342/Path_of_Exile_The_Awakening_Supporter_Packs_Achieve_RecordSales.php) put
the worldwide player base north of 10 million players which means on the order
of 0.01% of players compete in any given event.

The bump in the histogram at 15000 corresponds to the depth of the ladder
retrievable by the API. These events are mostly season-long leagues and the main
game's perennial Standard and Hardcore leagues. There is an outlier race that
was only 3 days long. The '3 Day Exiles Event HC (IC010)' was an in between
season event
that
[awarded closed beta access](http://www.pathofexile.com/forum/view-thread/1293370) to
the next expansion for the top 200 players. Additionally exile everywhere races
are especially unique, deadly (racers likely ended up having multiple
characters), and entertaining. In exiles everywhere, 20 rogue exiles
(mini-bosses) spawn per zone. You can check
out [ZiggyD](https://www.youtube.com/watch?v=t3TBU_Kwbes) racing this event to
see just how brutal these races are.

[TODO: Make race duration plot for leagues with 15,000 players]

Looking at how events have been distributed over time, we can see there are
generally 1000+ events a year. There is a noticeable up tick in events after
both the start of the open beta and the games official release.

[TODO: See if I can make a plot with the seasons overlaid. There are 16 seasons
listed here: http://pathofexile.gamepedia.com/Races]

![Events over time](/public/poe_events_over_time.png)

## Time series analysis

In any competitive game, the metagame is constantly in flux. Classes fade in and
out of popularity. After its release, the scion quickly became the most played
class. Over time interest in the scion has waned and in the past 12 months
players have started to favor racing rangers.

![Class popularity over time](/public/poe_class_popularity_vs_time.png)

[TODO: Plot of class wins over time]

## Player consistency

A common debate in games that have randomly generated maps, monsters, and loot
drops is whether players win based on skill or luck. Can you actually win
consistently given the random number generator (RNG)? Yes, yes you can. Look at
one of PoE's most famous racers Kripparrian.

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
compared to Kripps 259 in about 1 year. Since events aren't evenly distributed
but grouped into seasons, this means top racers are competing multiple times a
day, every day, during the month long race leagues.

And for fun let's look at professional Youtuber and streamer ZiggyD who races
regularly, but not quite at the level of the previous two players.

![ZiggyD race consistency](/public/poe_ziggyd_consistency.png)

Over 133 events ZiggyD has placed first overall 0%, top ten 1.5%, and top 100
31% of the time. So yes there is some RNG in Path of Exile, but the very best
racers can consistently break the top 100 in the majority of their races and top
ten more than a quarter of the time.

## Race difficulty

It seems clear that some races are more difficult than others: a Lethal mod race
where enemies do 50% extra normal damage and 50% normal damage as each element
should be more _lethal_ than a vanilla burst race. But quantifying race
difficulty is not immediately straightforward. If we look at the experience
histogram for characters in an event, it always has roughly an exponential
decay.

![Race experience histogram](/public/poe_experience_example.png)

There's a pile up of characters at low levels who die early on and then a long
tail of higher performers. I would claim that in the more difficult races
there's a larger disparity between these two populations. And this we can
quantify using the skew of the distribution. Specifically I used the adjusted
Fisher-Pearson standardized moment coefficient G1 built into most statistical
packages including pandas.

| League id                        |      Skew |
|----------------------------------|----------:|
| Void                             | 38.489430 |
| 6 Hour Cutthroat (C01E006)       | 24.641083 |
| 48 Hour Cutthroat (C01E010)      | 24.050988 |
| 12 Hour Cutthroat (C01E005)      | 20.607926 |
| 3 Day Exiles Event HC (IC010)    | 20.289373 |
| 1h BLAMT Party (FRW017)          | 20.133271 |
| 1 Week Cutthroat (IV008)         | 19.344031 |
| 5 Hour Cutthroat Party (S03F119) | 16.439814 |
| 2h BLAMT Party (MDC056)          | 16.046994 |
| 1 Hour FBLAMT Party (S06F169)    | 15.788544 |
| 2h BLAMT Party (MDC117)          | 15.630826 |
| 1h BLAMT Party (FRW011)          | 14.040574 |
| 1h BLAMT Party (ERW011)          | 13.756645 |
| SG_Void                          | 13.668209 |
| 4 Hour Cutthroat (C01E004)       | 13.176868 |
| 6 Hour Cutthroat (C01E008)       | 12.994884 |
| 3 Hour Cutthroat (C01E003)       | 12.877636 |
| 135M Lethal Rogue Solo (S03F124) | 12.583898 |
| SG_Invasion                      | 11.946815 |
| 24 Hour Endless Ledge (I018)     | 11.919397 |
| 1 Hr Exiles Everywhere (S10C013) | 11.658561 |
| 1 Hr Exiles Everywhere (S07F070) | 11.356278 |
| 6 Hour Cutthroat (C01E009)       | 10.940696 |
| 2Hr Lethal Rogue Party (S05F069) | 10.745195 |
| 1h BLAMT Party (ERW017)          | 10.695287 |
| 1 Hour BLAMT Solo (S03F042)      | 10.668457 |
| 90 Minute Lethal Party (S05F121) | 10.638022 |
| 1 Hour ELBLAMT (S07F151)         | 10.580864 |
| BLAMT Party (WHC139)             | 10.551859 |
| 3 Hour LI Party (S08F188)        | 10.508291 |
|                                  |       ... |
| SG_Solo Burst (STC032C)          |  0.256644 |
| 12 Min Burst (MDC034C)           |  0.254683 |
| SG_2 Min Solo Burst 2 (S03F091B) |  0.253730 |
| 12 Min Burst (MDC079C)           |  0.250269 |
| 12 Min Burst (MDC042B)           |  0.249229 |
| 12 Min Burst (MDC009C)           |  0.245536 |
| 12 Min Burst (MDC042C)           |  0.241469 |
| SG_2 Min Solo Burst 3 (S03F093C) |  0.233932 |
| SG_2 Min Solo Burst 2 (S02F040B) |  0.226202 |
| SG_Descent (STC002)              |  0.219688 |
| 12м Соло, Хет-трик (WHC010C)     |  0.209626 |
| SG_2 Min Solo Burst 3 (S04C039C) |  0.207356 |
| SG_Solo Burst (WHC034A)          |  0.204617 |
| SG_12 Min Burst (MDC025A)        |  0.188926 |
| SG_Solo Burst (STC032B)          |  0.151391 |
| SG_2 Min Solo Burst 2 (S04C066B) |  0.135552 |
| SG_Ledge Burst (STC037B)         |  0.128938 |
| SG_12 Min Burst (MDC025B)        |  0.126806 |
| SG_Solo Burst (WHC015C)          |  0.118678 |
| SG_2 Min Solo Burst 3 (S06F051C) |  0.092344 |
| SG_2 Min Solo Burst 3 (S03F072C) |  0.078516 |
| 12м Соло, Хет-трик (WHC062B)     |  0.042947 |
| SG_12 Min Solo Burst (BGC045A)   |  0.024881 |
| SG_2 Min Solo Burst 1 (S04C066A) |  0.007844 |
| More events                      |  0.000000 |
| SG_150m No-Proj Solo (S03C052)   |  0.000000 |
| SG_2 Min Solo Burst 2 (S04C095B) | -0.037703 |
| 1 Hour Endless Ledge (S07F004)   | -0.423922 |
| SG_Solo Burst (WHC034B)          | -0.561473 |
| SG_Endless Ledge (STC036)        | -1.244535 |


Let's digest this. Leagues with high skew include Void, Cutthroat, exiles
everywhere, and BLAMT. Leagues with low skew are mostly short vanilla Burst
races. Endless ledge seems to vary in skew possibly based on race length.

The top league is the Void league. Certain races allow characters access to rare
items or currencies at a much higher rate than normal (e.g. Descent races take
place in a dungeon completely separate from the main game with different drop
rates). Since moving these character's into the normal league would upset game
balance, these are moved into a special Void league. Once characters are in the
Void league they can only be viewed, not played. Since it is populated with
characters from the most "broken" leagues which can snowball (and from races
with different time lengths), it makes sense that it would be highly skewed.

Cutthroat centers around killing other players for loot and experience. Not
surprisingly most characters get killed at low levels while some snowball by
coming out on top early.

The 3 Day Exiles Event HC is the deadly exiles everywhere race discussed
earlier.

BLAMT is a perennial masochistic race the stands for **B**lood magic,
**L**ethal, **A**ncestral, **M**ulti-projectile, **T**urbo. This means
characters use health instead of mana for skills, monsters do 50% extra normal
damage and 50% normal damage as each element, there are many more enemey buff
totems, enemies fire four additional projectiles, _and_ monsters move, cast, and
attack 60% faster.

In contrast Burst races have no modifiers and the objective is to get as much
experience as possible in the time limit. It makes sense that there is a lot
less skew in experience for a 12 minute Burst race.

While it's pretty easy to say that BLAMT races are more difficult than Burst
races, it's unclear whether BLAMT is significantly more difficult than Exiles
Everywhere given this cursory analysis. That's one question we'll have to leave
for another post.

[TODO: Do a keyword search for BLAMT, Cutthroat, Exiles, Burst to see their average (maybe median would be best) skew]

[TODO: Write conclusion]
