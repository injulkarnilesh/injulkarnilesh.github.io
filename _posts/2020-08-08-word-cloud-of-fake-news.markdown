---
layout: post
title:  "Building Word Cloud of fake news debunked by Alt News for learning Python"
date:   2020-08-08 20:00:00
categories: python
tags: python data
image: /assets/article_images/2020-08-08-fake-news-word-cloud/fake-news.jfif
image2: /assets/article_images/2020-08-08-fake-news-word-cloud/fake-news.jfif
---
[Alt News](https://www.altnews.in/) is Indian fake news debunking site run by pretty liberal group of people. Thanks to high penetration of Facebook, Twitter, WhatsApp, cheap data rates, and non professional main stream media, India has become hotbed of fake news. These platforms are exploited for every possible purpose you can name, be it political propaganda, spreading communal hatred, pseudo-science etc.
Alt News does good job of debunking these fakes news by writing about them on their website. 

I recently had started learning python to improve my toolbox. To try some libraries I thought of peeking into what is involved into these fake news. 
I decided to crawl through the titles of the articles published on Alt News and build a Word Cloud to know who/what is involved more. 

Luckily, Alt News, built with WordPress, provides [RSS feed](https://www.altnews.in/feed/). RSS feed is usually used by news aggregator tools like Google Reader, Feedly etc. I decided to pull titles from this RSS feed, save them locally into a text file and then build a word cloud of that content.

Code for downloading titles from RSS feed:

<script src="http://gist-it.appspot.com/http://github.com/injulkarnilesh/python_tryout/blob/master/alt-news/alt-news.py"></script>

Here I am using
* requests: library to make HTTP calls
* xml: library to parse rss feed to extract titles.

End result of this is all titles saved into a file. As of 8th August 2020 content was [this](https://raw.githubusercontent.com/injulkarnilesh/python_tryout/master/alt-news/alt.txt).

Once I had the data, I used following code to create a word cloud from it:

<script src="http://gist-it.appspot.com/https://github.com/injulkarnilesh/python_tryout/blob/master/alt-news/cloud.py"></script>

For building the word cloud, I used `wordcloud` library.

I had to randomize the words because I did not want words appearing together to form new words.

Eg. If `congress mp` appeared multiple times together, the library would consider `congress mp` as different word than `congress mla`. What I wanted it to do was to consider `congress`, `mp`, `mla` as different words.

I used India's map as a mask so that cloud appears in the shape of India's map.

I also had to add some custom stop-words. Stop-words are generic non-value words like `is`, `are`, `the` etc. which should be excluded from counts considerations.

Resulting cloud was like this:
<img style="border: 2px solid black;" alt="Word Cloud" 
src="https://raw.githubusercontent.com/injulkarnilesh/python_tryout/master/alt-news/wordcloud-india.png"/>


### Disclaimer:
This was done with pure technical purpose of learning new stuff without any political intentions.
If you are offended by anything, please blame the `truth` not me.
