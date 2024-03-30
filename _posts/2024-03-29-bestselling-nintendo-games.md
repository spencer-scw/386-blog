---
layout: post
title: "Nintendo Bestsellers"
description: "What games released on Nintendo consoles have sold over a million copies?"
image: /assets/img/ultimate-oil.jpg
---

<p class="intro"><span class="dropcap">N</span>intendo is one of the most well-known game console designers and manufacturers in the world. As a fan of their products and games, I decided to curate a dataset of all of the games ever published on a Nintendo console that sold more than a million copies. I collected the data from Wikipedia list articles on the subject and cleaned it into an nice rectangular table. In this post, I'll go over the way I scraped the data from Wikipedia and offer insights to anyone attempting a similar project.</p>

## Exploring the data

The data I wanted to collect was available on Wikipedia. There is a seperate list article for bestsellers on every Nintendo console. You can check them out here:

1.    [NES](https://en.wikipedia.org/wiki/List_of_best-selling_Nintendo_Entertainment_System_video_games)
2.    [SNES](https://en.wikipedia.org/wiki/List_of_best-selling_Super_Nintendo_Entertainment_System_video_games)
3.    [N64](https://en.wikipedia.org/wiki/List_of_best-selling_Nintendo_64_video_games)
4.    [GC](https://en.wikipedia.org/wiki/List_of_best-selling_GameCube_video_games)
5.    [Wii](https://en.wikipedia.org/wiki/List_of_best-selling_Wii_video_games)
6.    [WiiU](https://en.wikipedia.org/wiki/List_of_best-selling_Wii_U_video_games)
7.    [Switch](https://en.wikipedia.org/wiki/List_of_best-selling_Nintendo_Switch_video_games)

![Screenshot of the Nintendo Switch List Article](assets/img/2024-03-29-bestselling-nintendo-games/image.png)

Right off the bat, one problem we might run into is that the columns are not consistently named or ordered in these tables. Another issue will be that some of the earlier consoles had sales reported in dollars, while the later consoles use are formatted in millions of dollars. We'll also want to add information to the data about which console the release was for. Altogether there are over 300 titles released for Nintendo consoles that sold more than a million copies. If we can get them all in the same dataset, there would be a lot of potential to examine them for patterns.

## Choosing a scraping method

I tried a handful of different things to get this data off of Wikipedia. I started off using the `requests` and `beautifulsoup` libraries to download and extract the table from the data. This turned out to be a clunky process, and Wikipedia kept rate-limiting me because I was trying to use a loop to grab all the data at once. Then, I tried using a wikipedia API and a wrapper someone wrote for Python, but unfortunately I kept having the same issue.

Eventually, I had to settle for scraping the pages one by one so that Wikipedia wouldn't rate-limit me. I also discovered that Pandas has a rudimentary built-in webscraping function, `read_html()` that actually uses `beautifulsoup` under the hood to search pages for table tags, and automatically loads them into dataframes. Perfect! Below is a snippet of code that worked for me:

```python
# Pandas takes care of this for me :) page_urls is a dictionary I defined that has all the urls to the articles.
NES_table = pd.read_html(page_urls["NES"])[2] # The table we need is the second one in the page.
```

This turned out to be surprisingly simple, and as long as I didn't scrape all 7 pages at once, Wikipedia didn't complain about my requests. I did this by scraping each page within a seperate code block in my notebook. If the table or tables you want to scrape are in a simple `<table>` tag on the website, I can't recommend this method enough; it's much more self-contained than a more manual solution. However, the `read_html()` function won't help you scrape data that's presented on a website in a more complicated format.

## Cleaning the data

Once I had all the tables read in seperately, I just needed to combine them all into one giant dataset. This largely consisted of 4 main steps for each dataframe:

1. Drop extra columns, like the column in Wikipedia for links to sources, or the `genre` column that only the newest consoles had.
2. Rename and reorder columns- to be combined, every table needed to have exactly the same column names and order.
3. Standardize number reporting for copies sold: convert the tables specifying this quantity in millions to be in dollars.
4. Add a column specifying which console the data came from

```python
# Dropping extra columns
Switch_table_cleaned = Switch_table.drop(["Genre(s)", "As of"], axis=1).rename(

    # Rename columns
    columns={"Title": "Game", "Copies sold": "Sales", "Release date[a]": "Release_Date", "Developer(s)": "Developer", "Publisher(s)": "Publisher"}

    # Reorder columns
)[["Game", "Developer", "Publisher", "Release_Date", "Sales"]]

# Convert the sales column to a raw integer
Switch_table_cleaned["Sales"] = Switch_table_cleaned["Sales"].apply(lambda num: int(float(num.split()[0].strip(">")) * 1000000))

# Console column
Switch_table_cleaned.insert(1, "Console", "Switch")
```

Once all that was done, it was a simple matter of concatenating all the tables and exporting the csv! That's it!

## Ethics

Data on Wikipedia is publicly and freely available. As long as you aren't spamming their servers with requests, they like to play nicely with scrapers and provide the information in an easy-to-parse format. These data don't contain any private or identifying information, so there aren't any ethical concerns with scraping and aggregating the information in these articles. Although I accidentally got rate limited early on, I tried my best to follow good scraping practice and didn't send too many requests too quickly.

## Conclusion

Check out the finished dataset here: [https://github.com/spencer-scw/nin-game-data](https://github.com/spencer-scw/nin-game-data)

There were a couple of obstacles to overcome in compiling this dataset, but I'm proud about how it turned out. Hopefully my experience proves helpful to anyone wanting to scrape Wikipedia tables like this. Stay tuned for a future article when I'll do some exploratory data analysis and look at some trends in the Nintendo bestsellers!