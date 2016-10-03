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
We created two spiders using Scrapy. First one named "top250_movie". This spider will go to the page [Top 250 rated movies from IMDB](http://www.imdb.com/chart/top?ref_=nv_mv_250_6) and extract information related to individual Url, Name and Year of each movie. Note: In order for you to successfully rerun the code, I expect you have already run throught the basic tutorial provided by [Scrapy](https://doc.scrapy.org/en/latest/intro/tutorial.html)

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
        		'year' : movie.css('span.secondaryInfo::text').extract()		}
{% endhighlight %}
