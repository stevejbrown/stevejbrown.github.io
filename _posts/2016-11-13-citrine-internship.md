---
layout: post
title: Citrine internship
---
    
# My summer at Citrine
This summer I had a data science internship at the materials startup [Citrine Informatics](http://www.citrine.io/).
Citrine aims to be the repository of the world's materials data. With this data the data science team builds
predictive models, optimization routines, and machine learning infrastructure that enable scientists and engineers to
solve their most pressing materials challenges.

There is always work to do at a startup! During the internship I built models for customer projects and demos, updated
our machine learning framework to Apache Spark 2.0, built datasets for over 30 different material properties spanning
everything from magnetism to crystal structure, put together
a [Cookiecutter](https://cookiecutter.readthedocs.io/en/latest/readme.html) template for new machine learning projects,
and wrote our pipeline cross-validation (with _much_ polishing by Erin Antono) among other infrastructure upgrades. One
of the highlights of my experience was that nearly everything I worked on made it into production code, helping me
feel like I was having an impact. Additionally there was a massive upgrade to our machine learning infrastructure over
the course of the summer and I was very fortunate to be part of the numerous high and low level discussions across
engineering and data science on how machine learning should be done, implemented, and exposed to end users on the
Citrine platform.


Building models and writing code as a team was different in many ways than toiling away on a PhD in seclusion. I was
very fortunate to work with a group of talented, helpful individuals who patiently taught me the ropes. Here are a few
of my salient lessons:

## Takeaways

### Time is your most valuable resource
This one is straight from co-founder Bryce Meredig. In a PhD there is a seemingly endless amount of time for more
experiments. Papers often get pushed back to flesh out more details and make the results iron clad. But at a startup,
while there are problems (and deadlines!) in abundance, time is in short supply. By their nature startups are on a fuse:
make it big before the money runs out. It is absolutely **crucial** to that you are working on the problems that
generate the most value for the company.

To work on the important problems you need to know what they are. For example, on one project I found myself sinking a
substantial amount of time into data cleaning. After a week of on-and-off work, I was gently asked what the hold up was.
It turned out that the majority of the value -- and what I was expected to be doing -- was just getting the first rough
cut of the data, not preserving every last data point. I immediately switched to whitelisting datasets and the project
got moving again. This is one of the main reasons why the data science and machine learning team at Citrine checked-in
briefly every morning: to make sure we were using our time efficiently and working on the high value-add problems.

### Consistency is critical
Each codebase has its own naming, style, and design conventions. As much as possible you want to keep these conventions
consistent across the company. Not only does it takes time to learn new conventions, but they fragment the codebase
making maintenance, upgrades, and interoperability challenging. This costs you time which, as previously mentioned,
is your most valuable resource.

Besides sharing consistent standards, I found templating to be a powerful way of keeping everything in sync. For someone
who is used to writing almost completely bespoke code for their entire PhD, templating may at first seem restrictive. It
_is_ a prescribed way of doing things, but that's exactly why it's so useful. Templated projects are built from standard
components and interfaces that are kept constant. This allows you to build, say, a new optimization routine for one
project and immediately expose that routine to every other project that you have built **and** teammates have built. You
can always deviate from the template if you need extra functionality, but starting from a template makes this the
exception rather than the rule. The [Cookiecutter](https://cookiecutter.readthedocs.io/en/latest/readme.html) project is
great for building your own templates and finding templates written by others (such as the
excellent [Cookiecutter Data Science](https://drivendata.github.io/cookiecutter-data-science/) template). Templating
proved so useful that my colleagues Sean Paradiso and Max Hutchinson built a full templating language for our machine
learning projects that replaced our simpler Cookiecutter template.

### Code review takes time (and it's worth it)
During my PhD I have been used to working on a personal codebase. There is no code review, except when I come back to do
a refactor. Moving to a shared codebase at Citrine, I knew I would be involved in code review, but I underestimated how
_long_ it would take. This summer I was spending at least an hour every day reviewing new pull-requests (and sometimes more
when my colleagues were especially productive :-)). To be honest, at first I was worried this would significantly slow
down my own contributions. But, like many good coding practices, you pay the price up front to reap the
rewards down the line.

Code review helps enforce consistency. I've already explained why consistency is so important -- it saves you time in
the future. It's tempting in the moment to check a fix or feature into version control, perhaps with an incomplete
docstring or a few tersely named variables, so you can move on to the next thing on your plate. Code review nips this in
the bug so the next time you (or your coworkers!) have to come back to your code you know what's going on.

A less apparent benefit of constant code review is learning how to write better code. Often I'd pick up new Scala
features and idioms by reading the incoming PRs (reducing using the \_+\_ syntax is so clean!). It is also a chance to
ask questions -- "Why did you write it this way?" and "How does this work?". This helps you get inside the head of the
author. Frequently I would gain a better understanding of the overall design and how everything was going to fit
together. With this understanding new code I wrote would more seamlessly integrate with everyone else's. By spending
more time reviewing _other_ people's code, I got to spend less time refactoring my own.

And yes, code review catches bugs. Lots of bugs. And it catches them early when they are easy to fix instead of later
when it can be a debugging headache.

### Make data provenance trivial 
Always make it very easy to track down where any part of the data comes from. Admittedly this should be best practice in
a PhD too, but it becomes doubly important when you are working with data sets that you didn't hand curate let alone
take all the data for. This means you should be able to say, "This one!" for any data point in a graph and immediately
find out where it came from. It is the data science equivalent of debugging code: you want to quickly find any problems.
Easy data provenance allows you to quickly identify whether outliers are erroneous (this point has a different crystal
structure) or truly exceptional (I should try making more materials like this).

To make this as easy as possible, I spent part of my time at Citrine adding the ability to display mouse-over text on
our plots so we could display, for example, the chemical formula or paper digital object identifier (DOI) for each
point. Previously we had to go back and dig into the individual records to extract this information. By shortcutting
this process, feedback cycles were tighter and iteration was faster which in the end led to more, better science getting
done. Additionally, exposing this kind of functionality to end users allows them to bring their own insights -- again,
better science. Try it with your own graphs! There are numerous plotting packages such as [Plotly](https://plot.ly/)
and [Bokeh](http://bokeh.pydata.org/en/latest/) with this functionality built in.
