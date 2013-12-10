---
layout: post
title: "Twiscala: Twitter Clone in Scala"
date: 2013-12-10 18:58
comments: true
categories: scala, technology
---
In this article I am going to talk about how to build a small twitter clone using Play Framework 2.1 , Scala as front end and Redis as backend. NetTuts+ has a very [good article]("http://net.tutsplus.com/tutorials/building-ribbit-in-scala/") on the same topic with Mysql as backend and redis.io also has a [case study]("http://redis.io/topics/twitter-clone") of building twitter clone using php as front end. However my motivation behind this post is to understand how data modelling in redis works . Since NetTuts+ already has a source code of frontend therefore I just treid to reuse the same.
<!-- more -->
Redis is an awesome and very popular k-v store or rather data structure store. Though k-v can be strings but Redis also supports data types like List, Set, OrderedList, SortedSet etc which makes it even more appealing.  To know more about redis visit [redis.io]("http://redis.io") and to know about typical use case visit [common-web-use-cases-solved-in-redis]("http://highscalability.com/blog/2011/7/6/11-common-web-use-cases-solved-in-redis.html").
Coming from RDBMS background, takes a while in understanding the k-v store, specially data modelling.  Let's go step by step for twiscala app...
<p>
User and Authentication
{% codeblock Account.scala lang:scala %}
 RedisDB.withClient {
        client =>
          val uid = client.incr("global:nextuid").get
          client.set("uid:" + uid + ":email", email)
          client.set("uid:" + uid + ":name", name)
          client.set("uid:" + uid + ":password", BCrypt.hashpw(password, BCrypt.gensalt()))
          client.set("email:" + email + ":uid", uid)

      }
{% endcodeblock %}
Here we are using global:nextuid key to get the next available value of uid (similar to auto increment pk value in rdbms) and later all the attributes are being set based on uid: :attribute based key so that if one has the uid , all the attributes are easily retreivable.
Last key in the code above has been set to make uid available by email id.
</p>
<p>
Ribbit (Tweets)
{% codeblock Account.scala lang:scala %}
RedisDB.withClient {
      client =>
            val postId = client.incr("global:nextRibbitId").get
            println(postId)
           client.set("ribbit:" + postId + ":post", sender.get + "|" + DateTime.now() + "|" + content)
            val account = Account.findByEmail(sender.get).get

            val followers = client.smembers("uid:" + account.id + ":followers")
            if (followers != null && followers.isDefined) {
              for (follower <- followers) {
                client.lpush("uid:" + follower + ":posts", postId)

              }
              client.lpush("global:timeline", postId)
              client.ltrim("global:timeline", 0, 1000)
            }

        }
 {% endcodeblock %}
 See carefully how ribbit key has bee constructed and how we are using List to keep all the ribbit  in order of the time , most recent to old.
</p>
The application is just a POC to learn few  preliminary things about redis therefore I have not implemented functionality like followers and following. Sets  data structure can be used to keep track of followers and following people. For more detail do visit redis case study link given earlier. Entire source code of this application can be downloaded from [this location]("https://github.com/pkmishra/scala-playground/tree/master/twiscala"). Hopefully this will create  at least some curiosity about redis use cases.
