---
layout: post
title: "How to show google mail like loading image/please wait… message while page data loads"
date: 2008-04-13 23:11
comments: true
categories: techtips
---
For a better user experience you would your users to see please wait message while browser render the page completely. Here is the one solution to the same problem.

Let’s create a master page called Site.master and a web content form as demo.aspx.
<!-- more -->
{% codeblock DemoObject lang:html %}
<title>Loading Demo</title>
    <form>
 
<div>
 
<!-- Page Header will go here... -->
 
</div>
 
<div>
 
 
                                            <!-- Page-specific content will go here... -->
                                         
 
</div>
 
<div>
 
<!-- Page Footer will go here... -->
 
</div>
{% endcodeblock %}

Demo.aspx is the web content form which fetches data from a database.
To show loading message add following code to the code behind of master file Site.master.cs

{% codeblock  lang:csharp %}
protected override void OnLoad(EventArgs e)
  {
  if (!IsPostBack)
  {
  Response.Buffer = false;
  Response.Write(” Please wait…“);
  Response.Flush();
  }
  base.OnLoad(e);
  }
  protected override void Render(HtmlTextWriter writer)
  {
  if (!IsPostBack)
  {
  Response.Clear();
  Response.ClearContent();
  }
  base.Render(writer);
  }
{% endcodeblock %}

in the Site.master.aspx file add following javascript at the end of the file.
{% codeblock  lang:javascript %}
 try{
 var divLoadingMessage =  document.getElementById("divLoadingMsg")
 if (divLoadingMessage != null && typeof(divLoadingMessage) != 'undefined')
        {
            divLoadingMessage.style.display="none";
            divLoadingMessage.parentNode.removeChild(divLoadingMessage);
        }
    }catch(e){}
{% endcodeblock %}

That’s it now all your pages using Site.master will be showing Please wait.. message when the page starts loading. Of course instead of putting a message you can put a nice web2.0 loading image in between divLoadingMsg tags.
So how does it work?
The onLoad event of master page will be called before any of the content web form’s Onload event. as soon as master page loads div tag becomes visible. After the page has loaded completely the script written at the end of the master page hides the div tag. so simple isnt’t it.Hope this helps!
