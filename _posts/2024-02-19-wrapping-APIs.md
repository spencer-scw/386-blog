---
layout: post
title: "Wrapping APIs"
description: "How to write a tool that wraps and extends an API in Python"
image: /assets/img/connections.jpg
---

<p class="intro"><span class="dropcap">A</span>PIs are very important tools for doing data science. The first step in doing any kind of statistical analysis is to collect the data. As data scientists, we need to be well versed in all of the methods for collecting and cleaning data. In this tutorial, I will walk through my process of taking an API for a database of transcripts and creating a command-line tool that allows you to collect data. As a bonus, it's a fun little command that I can use to customize my Linux terminal!


# First steps


So, you've found an API you'd like to use. The first thing to do is to explore it and learn how it works. Does it have documentation? Is there a web interface that will help you to get familiar with the endpoints? What format does the data come in? Will it need cleaning?

For this tutorial I'll be using the [WOB api](https://wob.coppermind.net/api/). WOB stands for Words of Brandon, which is the community name for a collection of statements and answers by fantasy author Brandon Sanderson, which have been transcribed and saved in a database.

![WOB website](/assets/img/2024-12-19-wrapping-APIs/WOB-screenshot.png)

This API offers six different endpoints:


1. /events
2. /upcoming_events
3. /tags
4. /entry
5. /search_entry
6. /random_entry


Each of these endpoints are used for different purposes, some of them internal to running the website. /events and /upcoming_events will show you events at which Brandon has spoken or will be speaking at. /tags is for getting quotes about certain topics. /entry is for getting specific entry ids, and /search_entry is for searching entries by keyword, tag, speaker, and date. The last one, /random_post is of particular interest to me as a data scientist, because I can use it to get a simple random sample of posts and create a dataset to analyze.


Let's take a look at what we get when we use the /random_post endpoint:


```bash
curl https://wob.coppermind.net/api/random_entry/?format=json
```

returns


```json
{
    "id":8128,
    "event":171,
    "event_name":"Oathbringer release party",
    "event_date":"2017-11-13",
    "event_state":"N/A",
    "date":"2017-11-13",
    "paraphrased":false,
    "modified_date":"2018-01-31T19:20:31.158393Z",
    "tags":["shardblades"],
    "lines":[
        {"speaker":"Questioner",
        "text":"<p>So, a Radiant's blade. When it takes other forms, does it take on any different properties? So, like, if Kaladin beat someone with the butt of his Sylspear, would it still do something in regards to the soul?</p>"},
        {"speaker":"Brandon Sanderson",
        "text":"<p>Would he still hit the soul? That is theoretically possible to make happen. It requires a lot of work. That is theoretically possible.</p>"}],
    "note":""
}
```

It looks like the response object for the random post endpoint gives us a bunch of metadata and then the actual content of the quote. (this one is in question and answer format, as are a lot of them. Many of these quotes come from Q and A sessions with Brandon.)

There's just one main problem though: It looks like we've still got to deal with stray \<p> and other HTML tags. Evidently whoever set up this API left them in there. For our purposes though, it would be nice if we could clean them up. That's where some webscraping skills come in handy.


# Writing the Wrapper


I decided to write my wrapper in python. I'm going to use two external libraries: beautifulsoup and requests. The first thing to do is import the packages:

```python
import requests
import sys
from bs4 import BeautifulSoup
```

We'll need to poll the API using requests:

```python
r = requests.get('https://wob.coppermind.net/api/random_entry/?format=json')
```

And finally we can use beautifulsoup to clean up those HTML tags.

I've written two different forms of output. One is just the plain text: we grab the ["lines"] object from the JSON and print it to the terminal:

```python
text_lines = [line for line in r.json()["lines"]]
    for line in text_lines:
        soup = BeautifulSoup(line["text"], 'html.parser')
        print(f'{line["speaker"]}: {soup.get_text()}')
```

The output for this "pretty printing" format looks something like this:

```
Questioner: So do Inquisitors, when they use Allomancy, have to actually ingest the metals?
Brandon Sanderson: They actually do. So what is happening is Hemalurgy rips off a piece of one person's soul and spikes it somebody else and so it is basically taking off the piece of someone's soul that makes them an Allomancer and adding it to someone else instead and so then they act as an Allomancer just as it would happen.
```

I've also written very similar code that returns a JSON object with the HTML tags cleaned. I plan to do some exploratory data analysis of a dataset I create with this API later on.


# Conclusion


It was a lot of fun for me to mess around with this API. I learned that it doesn't have very good documentation right now, so I'm actually writing some to contribute to the project. Every API is going to be different. This one happened to have the added challenge of the HTML tags in it that needed to be cleaned up. The key to making a good wrapper is to explore the functionality of the API, and make it easy to use the features that you're interested in.

Try it out on your own! The sky is the limit. Plus, if you do it right, you can enjoy the results in your own terminal too! (cow not required.)

```
 _________________________________________
/ Questioner: What’s your most terrifying \
| character, and why is it Nightblood?    |
|                                         |
| Brandon Sanderson: Ha. Nightblood is    |
| pretty terrifying… You know, an object  |
| created to destroy evil but doesn’t     |
| know what it is?                        |
|                                         |
| Questioner: When you brought it         |
| back...I had <to like shut the book> a  |
| little bit and like, scary.             |
|                                         |
| Brandon Sanderson: I wanted you to      |
| think that Nightblood in the hands of   |
| Szeth should be one of the most         |
| terrifying things that you have ever    |
\ contemplated.                           /
 -----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```