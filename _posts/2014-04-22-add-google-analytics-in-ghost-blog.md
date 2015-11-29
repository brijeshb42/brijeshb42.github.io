---
color: blue
layout: post
title:  "Add Google Analytics in your ghost blog"
date:   2014-04-22 19:20:35
categories: 
tags: [ghost, blog, google, analytics]
---

Ghost blogging system is very new and highly under development. It does not support plugins yet but will soon support in future versions. Untill then, if we want to add Google Analytics to our blog, we have to add the tracking code manually to its theme files.

* To add analytics to your blog, first signup at ```https://www.google.com/analytics```.
* After signing up, you will have to add and verify that you own the site.
* After verification, you will be provided with a javascript code snippet like this:

{% highlight html linenos %}
<script>
	(function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
		(i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
		m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
	})(window,document,'script','//www.google-analytics.com/analytics.js','ga');
	ga('create', 'UA-123456789-1', 'bitwiser.in');
	ga('send', 'pageview');
</script>
{% endhighlight %}

* Copy the code snippet.

* Browse to the theme directory you are using in your ghost installation folder. Assuming the directory id ```ghostblog/content/themes/casper```.
* In this directory, there will be a file named ```default.hbs```.
* Open the file and paste the code just above the ending ```</body>``` tag.
* This will make sure that analytics is available in each and every page of your blog.
* Save the file and push the changes to your web host.
* Analytics is now added and now you can visit google analytics and track your website visitors.
