---
layout: post
title: CS638 Data Science
tags: [Data science, machine learning, web crawling]
---

#### Backgroud
I am taking Computer Science 638 - Data Science from University of Wisconsin Madison this semester. I will share the course information here including the course project we did. [Course Website](https://sites.google.com/site/anhaidgroup/courses/cs-638-fall-2016)

#### Project
The project is about crawling data from websites, extracting useful features, joining data from different sources, and finally building some machine learning models. We primarily use python. The project is being divided into several stages.

* Stage 0: Form a team. Due Tue Sept 20
* Stage 1: Crawl and extract to obtain two tables, three weeks, due Tue Oct 11
* Stage 2: Explore, understand, clean, transform the two tables


#### Detail


##### Stage 1
We decided to use scrapy and beautiful soup from Python as our primary packages to first collect the data we need. We are interested in movie related websites such as IMDB and Yahoo movie. 

##### Group meeting Oct 01 :  
I created two spiders using Scrapy. First one named "top250_movie". This spider will go to the page [Top 250 rated movies from IMDB](http://www.imdb.com/chart/top?ref_=nv_mv_250_6) and extract information related to individual Url, Name and Year of each movie. Note: In order for you to successfully rerun the code, I expect you have already run throught the basic tutorial provided by [Scrapy](https://doc.scrapy.org/en/latest/intro/tutorial.html)

Below code defines a spider named "top250_movie", the starting url is "start_urls" and for each part of the page that satisfy the structure I defined, extracts the data that is related to "url", "name" and "year".  

{% highlight javascript linenos %}
import scrapy

class Top250_movie(scrapy.Spider):
	name = "top250_movie"
	start_urls = ['http://www.imdb.com/chart/top?ref_=nv_mv_250_6']
	
	def parse(self,response):
	    for movie in response.css('td.titleColumn'):
		yield {
			'url' : movie.css('a::attr(href)').extract(),
        		'name' : movie.css('a::text').extract(),
        		'year' : movie.css('span.secondaryInfo::text').extract()	
			}
{% endhighlight %}

Now, we have been able to crawl the information from one single page. Wouldn't it be interesting to iteratively go into each specific links (eg, each movie link from the Top 250 movie page) and extract information within those pages?

Below codes create a spider named "loopeachmovie" and used the same method before to get the link of each movie from the starting url "start_urls". For each individual movie link it acquired, it will send a Request using the new link and a callback funtion defined by "parse_movie". "parse_movie" is the structure I defined here to extract the data within each individual movie page. Essentially, for each selectd url, Scrapy will create a Response object and you can extract data you want using Response's methods such as its CSS and Xpath functions.

{% highlight javascript linenos %}
import scrapy

class loopeachmovieSpider(scrapy.Spider):
	name = 'loopeachmovie'
	start_urls = ['http://www.imdb.com/chart/top?ref_=nv_mv_250_6']
	
	def parse(self,response):
	    # follow links to each movie page from top250 rated movie
	    for href in response.css('td.titleColumn').css('a::attr(href)').extract():
		yield scrapy.Request(response.urljoin(href),
				     callback = self.parse_movie)
				     
	def parse_movie(self,response):
	    
	    yield {
		'name' : response.css('div.title_wrapper').css('h1::text').extract()[0],
		'year' : response.css('div.title_wrapper').css('span a::text').extract(),
		'length' :response.css('div.title_wrapper').css('div.subtext').css('time::text').extract() 
		}
{% endhighlight %}

If you want to quickly understand what is the result of say 
{% highlight javascript linenos %}
response.css('div.title_wrapper').css('h1::text').extract()[0]
{% endhighlight %}
I suggest you run the following code in your terminal, 

```
scrapy shell 'http://www.imdb.com/title/tt0111161/?pf_rd_m=A2FGELUUNOQJNL&pf_rd_p=2398042102&pf_rd_r=0M6MJZ0HKHS1G28630ZJ&pf_rd_s=center-1&pf_rd_t=15506&pf_rd_i=top&ref_=chttp_tt_1'
```
this will open an interactive shell with an object named response created by the link you provided. This is what happened internally from the loopeachmovie spider, when it acquire the inidivual movie link and create the associated Response object.
{% highlight javascript linenos %}
response.css('div.title_wrapper').css('h1::text').extract()[0] # this returns the name of the movie
response.css('div.title_wrapper').css('span a::text').extract() # year of movie
response.css('div.title_wrapper').css('div.subtext').css('time::text').extract() # length of the movie
{% endhighlight %}
You can go to link's source page and extract other data you want following the same template.


##### Group meeting Oct 09 :
Now, We want to crawl a large amount of movies' information from two different movie websites. I am responsible for crawling 8000 movies from IMDB. The first problem that came into my mind is how can I get the links for those 8000 movies? Based on some initial exploration, IMDB does not provide a simple page listing all the movies in the histroy. Luckily, I found a [post](http://www.imdb.com/list/ls057823854/?start=001&view=detail&sort=listorian:asc) provided by a user in IMDB. As you can see from the post, it provides links to individual movie all the way back to 1972. Nice surprise ha? Each page of the post contains 100 movies. What I need to do next is figure out a way to automatically go to next page. One simple way I came up with was by observing the page url.  
http://www.imdb.com/list/ls057823854/?start=001&view=detail&sort=listorian:asc  

You can easily find out the pattern, which is in "?start=???". Thus, what I need to do in python is that I just need to create a list, containing all the url from start=001 to start=7900 and then assign them to start_urls.  Following this setup, the spider is able to iteratively loop through each page, collect the 100 movies link, go into each link and crawl the required data.  
{% highlight javascript linenos %}
index = range(1,8000,100)
start_urls = list()
for page in index:
    	start_urls.append('http://www.imdb.com/list/ls057823854/?start=' + str(page) + '&view=detail&sort=listorian:asc')
{% endhighlight %}
One might ask why you don't use other Scrapy functionality such as the CrawlSpider, Rule, and LinkExtractor, so that you do not need to preprovide the required links. Instead, you can let the spider to extract the link based on some regular expressions ? The answer is Yes, this is definitely the right way to go and is something I will try in the future.  
Different from previous procedures, this time I also want to store the html page of each movie I crawled. Therefore, I add a several lines of code in the spider, which use another python package Urllib2. 
{% highlight javascript linenos %}
def parse(self,response):
	 
	    # follow links to each movie page from top250 rated movie
	    for href in response.css('div.info').css('b a::attr(href)').extract():
		# Use urllib2.urlopen to open and read each movie page
		page = urllib2.urlopen('http://www.imdb.com/' + href)
		page_content = page.read()
		# Define unqiue name for each page based on the uniqe letter from movie url
		unique_name = "html_pages/" + href.split("/")[2] + ".html"
		# Open a html file on local disk and write the html page out.
		with open(unique_name, 'w') as page:
		     page.write(page_content)
        
		yield scrapy.Request(response.urljoin(href),
				     callback = self.parse_movie)
{% endhighlight %}

After add in all the data I want from each movie page, I finally have the following spider.
{% highlight javascript linenos %}
import scrapy
import urllib2
import os
class loopeachmovieSpider(scrapy.Spider):
	name = 'allmovie_IMDB'

	index = range(1,8000,100)
	start_urls = list()
	for page in index:
    		start_urls.append('http://www.imdb.com/list/ls057823854/?start=' + str(page) + '&view=detail&sort=listorian:asc')

	def parse(self,response):	 
	    # follow links to each movie page from top250 rated movie
	    for href in response.css('div.info').css('b a::attr(href)').extract():
		page = urllib2.urlopen('http://www.imdb.com/' + href)
		page_content = page.read()
		unique_name = "html_pages/" + href.split("/")[2] + ".html"
		with open(unique_name, 'w') as page:
		     page.write(page_content)
        
		yield scrapy.Request(response.urljoin(href),
				     callback = self.parse_movie)
				     
	def parse_movie(self,response):
             yield {
		'name' : response.css('div.title_wrapper').css('h1::text').extract()[0],
		'year' : response.css('div.title_wrapper').css('span a::text').extract(),
		'length' :response.css('div.title_wrapper').css('div.subtext').css('time::text').extract(),
        	'director':response.css('div.credit_summary_item')[0].css('a span::text').extract(),
        	'writers':response.css('div.credit_summary_item')[1].css('span a span::text').extract(),
        	'stars':response.css('div.credit_summary_item')[2].css('span a span::text').extract(),
        	'genre':response.css('div.subtext')[0].css('a[href*=genre] span::text').extract(),
        	'Description':response.css('div.plot_summary').css('div.summary_text').css('div::text').extract()
		}
{% endhighlight %}

My teammate also crawl 6000 movie information from Rotten Tomato, which primarily use urllib. The structure and idea is basically the same. You can have a [look](https://github.com/yaluai/Scrapy_Rotten_tomatoes/blob/master/main.py) if you want.

Now, we have movie information from two websites and is time to go to next stage where we will perform some data exploration, cleaning, understanding and transforming. Below are the table we have.

[IMDB movie 8000 html](https://drive.google.com/file/d/0B93R4jyn0EfRRDREU3prVlpXVjg/view)
[IMDB movie 8000 csv](https://drive.google.com/file/d/0B93R4jyn0EfRdDNvVC1YMkF5aU0/view)
[Rotten Tomato 6000 html](https://drive.google.com/open?id=0B-LRCz94qwYEbDhTS2VQeHh5OG8)
[Rotten Tomato 6000 csv](https://drive.google.com/open?id=0B-LRCz94qwYEbGJIUnExUjJ1Qnc)

##### Stage 2
Coming next...
