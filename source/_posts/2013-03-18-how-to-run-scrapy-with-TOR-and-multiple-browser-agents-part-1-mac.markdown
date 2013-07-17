---
layout: post
title: "Scrapy: run using TOR and multiple agents"
date: 2013-03-18 20:02
comments: true
categories: techtips
---
[Scrapy](http://scrapy.org) is a brilliant and well documented crawler written in python. Though it is not as scalable as Apache Nutch but it can easily handle thousands of sites easily. You can get up and running very quickly using the official documentation.
[Tor](https://www.torproject.org/) gives you power to keep your privacy and security.Tor can hide you so that website can not track your identity. You may read more about TOR in [official site](https://www.torproject.org/about/overview.html.en). However Tor only works for TCP streams and can be used by any application with SOCKS support.
<!-- more -->
When we combine Scrapy with Tor, we can have more control over our crawler privacy. We already know that Scrapy can work with proxy server however since Scrapy doesn't work directly with SOCKS proxy, things can work out if we can introduce a http proxy server as an intermediate between Scrapy and Tor which can also speak to Tor using SOCKS. SOCKS protocol is a lower level protocol than http and it is more transparent in a sense that it doesn't add extra info like http-header etc.  We are going to use a tiny and fast proxy server [polipo](http://www.pps.univ-paris-diderot.fr/~jch/software/polipo). Polipo can talk to Tor using SOCKS protocol therefore all three together can work to create anonymous crawler. 
Alright let's get started. 

+ I am going to assume that you have already installed scrapy on your system. 
+ Install Tor as per the instruction from [official documentation](https://www.torproject.org/docs/documentation.html.en). On my mac I used macport to install Tor.
+ Start tor
+ Install polipo using macport.
+ uncomment following lines in /etc/polipo/config or /opt/local/etc/polipo/config file. <br />
{% codeblock build.sbt lang:sh %}
socksParentProxy = localhost:9050
diskCacheRoot=""
disableLocalInterface="" 
{% endcodeblock %}
+ start polipo.
By default polipo listens on 8123 port and Tor on 9050 port. If you want you may change this port and accordingly adjust settings in config files.

Now to verify if everything is working fine
+ change your broser proxy setting to point to localhost and port 8123.
+ Visit [this tor check page](https://check.torproject.org). This page should give your message that you are using tor correctly depending upon if everything is configured correctly.

Now after basic setup is complete let's add middleware code in Scrapy to make use of this proxy.
+ Add a new file called middlewares.py in your project and add following code.[See this Gist](https://gist.github.com/pkmishra/5193452)

{% codeblock middlewares.py  lang:python %}
import os
import random
from scrapy.conf import settings
class RandomUserAgentMiddleware(object):
    def process_request(self, request, spider):
        ua  = random.choice(settings.get('USER_AGENT_LIST'))
        if ua:
            request.headers.setdefault('User-Agent', ua)
 
class ProxyMiddleware(object):
    def process_request(self, request, spider):
        request.meta['proxy'] = settings.get('HTTP_PROXY')
{% endcodeblock %}

+ In settings.py add the code shown below. [See this Gist](https://gist.github.com/pkmishra/5193452)
{% codeblock settings.py lang:python %}
### More comprehensive list can be found at 
### http://techpatterns.com/forums/about304.html
USER_AGENT_LIST = [
    'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.7 
         (KHTML, like Gecko) Chrome/16.0.912.36 Safari/535.7',
    'Mozilla/5.0 (Windows NT 6.2; Win64; x64; rv:16.0) 
       Gecko/16.0 Firefox/16.0',
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_3) AppleWebKit/534.55.3 
       (KHTML, like Gecko) Version/5.1.3 Safari/534.53.10'
]
HTTP_PROXY = 'http://127.0.0.1:8123'
DOWNLOADER_MIDDLEWARES = {
     'myproject.middlewares.RandomUserAgentMiddleware': 400,
     'myproject.middlewares.ProxyMiddleware': 410,
     'scrapy.contrib.downloadermiddleware.useragent.UserAgentMiddleware': None
    # Disable compression middleware, so the actual HTML pages are cached
}
{% endcodeblock %}

You are all set to crawl now. 
<b>Make sure you use the crawler responsibly with sufficient delay and follow the website terms and conditions along with robots.txt rules.</b>

Next time I am going to setup things on ubuntu and accordingly I will update this article.
Hope this helps!!!