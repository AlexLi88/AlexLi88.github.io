---
title: "A Web Crawler I Recently Did"
tags: [python, crawler]
categories: [programming]
comments: [true]
---
These days, I have benn working on some web crawlers with Scrapy and BeautifulSoup, which are both very helpful open sourced tools for web crawling in Python. 

First of all, I would like to introduce what the Web Crawler is.  

> A Web crawler is an Internet bot which systematically browses the World Wide Web, typically for puposed of Web Indexing. [Web crawler](https://en.wikipedia.org/wiki/Web_crawler)

Crawling is not getting data from the web programmatically, but is actually use a "bot" to find the URL and download the data you want. I will use an example to illustarte how does a real web crawler works in detail. 

Last week, my supervisor gave me a task to download 15 year's data of Leaf Area Index on [NASA NEO](http://neo.sci.gsfc.nasa.gov/view.php?datasetId=MOD15A2_M_LAI),and he want .csv files which are 0.5 degree and 720*360. 

After analyzing the download URL on [NASA NEO](http://neo.sci.gsfc.nasa.gov/view.php?) you can easily find all the monthly data follow a same pattern, and the only difference is that they have an unique SI number to indicate which month of the data is.  

- http://neo.sci.gsfc.nasa.gov/servlet/RenderData?si=`1700072`&amp;cs=rgb&amp;format=CSV&amp;width=1440&amp;height=720
- http://neo.sci.gsfc.nasa.gov/servlet/RenderData?si=`1698735`&cs=rgb&format=JPEG&width=1440&height=720
- http://neo.sci.gsfc.nasa.gov/servlet/RenderData?si=`1697617`&cs=rgb&format=JPEG&width=1440&height=720

So far, the only thing we need to figure out is how do we find out the SI number of each month.  If you check the page source you will find a timeline div which holds all the monthly information for each year. 

```html
<div id="timeline" class="overview timeline-month">
        <div class="slider-elem month" id="2015-01-01"><a onclick="viewDataset('1611576','2015-01-01');" href="javascript:void(0);">January 2015</a></div>
        <div class="slider-elem month" id="2015-02-01"><a onclick="viewDataset('1612886','2015-02-01');" href="javascript:void(0);">February 2015</a></div>
        <div class="slider-elem month" id="2015-03-01"><a onclick="viewDataset('1614569','2015-03-01');" href="javascript:void(0);">March 2015</a></div>
        <div class="slider-elem month" id="2015-04-01"><a onclick="viewDataset('1615527','2015-04-01');" href="javascript:void(0);">April 2015</a></div>
        <div class="slider-elem month" id="2015-05-01"><a onclick="viewDataset('1616994','2015-05-01');" href="javascript:void(0);">May 2015</a></div>
        <div class="slider-elem month" id="2015-06-01"><a onclick="viewDataset('1656779','2015-06-01');" href="javascript:void(0);">June 2015</a></div>
        <div class="slider-elem month" id="2015-07-01"><a onclick="viewDataset('1691485','2015-07-01');" href="javascript:void(0);">July 2015</a></div>
        <div class="slider-elem month" id="2015-08-01"><a onclick="viewDataset('1694155','2015-08-01');" href="javascript:void(0);">August 2015</a></div>
        <div class="slider-elem month" id="2015-09-01"><a onclick="viewDataset('1695153','2015-09-01');" href="javascript:void(0);">September 2015</a></div>
        <div class="slider-elem month" id="2015-10-01"><a onclick="viewDataset('1696373','2015-10-01');" href="javascript:void(0);">October 2015</a></div>
        <div class="slider-elem month" id="2015-11-01"><a onclick="viewDataset('1697617','2015-11-01');" href="javascript:void(0);">November 2015</a></div>
        <div class="slider-elem month" id="2015-12-01"><a onclick="viewDataset('1698735','2015-12-01');" href="javascript:void(0);">December 2015</a></div>
</div>
```

Then the next step is to use Beautifulsoup to pull data out of HTML and download the file to loacl. 

```python
import urllib2 as url
from bs4 import BeautifulSoup
def getIs():
    link = "http://neo.sci.gsfc.nasa.gov/view.php?datasetId=MOD15A2_M_LAI&year="
    dl_link = "http://neo.sci.gsfc.nasa.gov/servlet/RenderData?si=XXX&cs=rgb&format=CSV&width=720&height=360"
	all_is = []
	for year in range(2000,2016):
		
		year_link = link + str(year)
		pageFile = url.urlopen(year_link)
		pageHtml = pageFile.read()
		pageFile.close()

		soup = BeautifulSoup("".join(pageHtml))
		divs = soup.find_all("div", class_="slider-elem month")
		for div in divs:
			is_dic = {}
			month = div['id']
			text = div.find("a")
			onclick = text['onclick']

			i0 = onclick.index("(")
			i1 = onclick.index(",")
			is_dic['is'] =  onclick[(i0+2): (i1-1)] 
			is_dic['month'] = month
			all_is.append(is_dic)

	return all_is

def dl_file(all_is):
	for item in all_is:
		file_name = item['month'] + ".csv"
		completeName = os.path.join(csv_path, file_name)
		link = dl_link.replace("XXX", str(item['is']))
		f = url.urlopen(link)
		print "Downloading: " + file_name
		with open(completeName,"wb") as code:
			code.write(f.read())

		print "Download " + file_name + " finished! "

		f.close()
```
After you get all the csv files you can convert the ascii files to Netcdf format, and then you can use some tools like [Ncview] to get a geographically of these files. 

Final Results:
![Sample output: ](/assets/post_img/2016_03_25_sample_output.gif) 


