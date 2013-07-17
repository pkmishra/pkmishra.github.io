---
layout: post
title: "Scrapy: run using TOR and multiple agents Part 2"
date: 2013-04-16 21:22
comments: true
categories: techtips
---
As discussed in [last post]("http://pkmishra.github.io/technical/2013/03/18/how-to-run-scrapy-with-TOR-and-multiple-browser-agents/") this post is about running the same things on ubuntu. Again I am going to assume that you already have scrapy installed on your system. To install Tor

<!-- more -->
+ Add tor repository to your ubuntu repository list (See [official documentation]("https://www.torproject.org/docs/debian.html.en#ubuntu") for more detail)

{% codeblock lang:sh %}
echo "deb  http://deb.torproject.org/torproject.org quantal main" | sudo tee -a /etc/apt/sources.list
{% endcodeblock %}

  Here quantal is the release name for Ubuntu 12.10 release. If you are running any other version of ubuntu change it accordingly.
+ add gpg keys to sign the package


{% codeblock  lang:sh %}
gpg --keyserver keys.gnupg.net --recv 886DDD89
gpg --export A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89 | sudo apt-key add -
{% endcodeblock %}

+ Now referesh the package and install tor

{% codeblock  lang:sh %}
apt-get update
apt-get install deb.torproject.org-keyring
apt-get install tor
{% endcodeblock %}


+ Install Polipo install using apt-get install polipo and change the configuration as described in last post. Everything else remains same i.e. by default polipo is using 8123 and Tor 9050 so you don't have to modify anything. 
+ By default Tor and polipo will be installed as service. So after changing configuration you must restart polipo service 

{% codeblock lang:sh %}
using sudo /etc/init.d/polipo restart 
(you can use same comand to start/stop both the services)
{% endcodeblock %}


+ Now check whether tor is working fine or not by running 

{% codeblock lang:sh %}
curl --proxy http://localhost:8123 https://check.torproject.org/ | grep "Congratulations. Your browser is configured to use Tor"
{% endcodeblock %}


You can see you will get this text displayed if everything is working. You are all set now.
Hope this helps