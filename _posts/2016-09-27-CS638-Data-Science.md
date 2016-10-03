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



#### Detail


##### Stage 1
We decided to use scrapy and beautiful soup from Python as our primary packages to first collect the data we need. We are interested in movie related websites such as IMDB and Yahoo movive. 

Group meeting Oct 01 :  
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

What is next ? We might experiment using the regular expression and allow the spider not simply go into one next page. Instead, go into all the pages you can as long as it satisfied the format we defined.
