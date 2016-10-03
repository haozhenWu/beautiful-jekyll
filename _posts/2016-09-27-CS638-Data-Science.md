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

Now, we have been able to crawl the information from one single page. Wouldn't it be interesting to iteratively go into specific links (eg, each movie link from the Top 250 movie page) and extract information within those pages?

Below codes create a spider named "loop eachmovie" and used the same method before to get the link of each movie from the starting url "start_urls". For each individual movie link it acquired, it will send a Request using the new link and a callback funtion defined by "parse_movie". "parse_movie" is the structure I defined here to extract the data within each individual movie page. Essentially, for each selectd url, Scrapy will create a Response object and you can extract data you want using Response's methods such as its CSS and Xpath functions.

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
