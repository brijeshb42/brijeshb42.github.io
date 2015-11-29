---
layout: post
color: grey
title:  "Add comment support in your Ghost Blog"
date:   2014-04-22 19:16:35
categories: 
tags: [ghost, blog, comment, disqus]
---

[Ghost](https://ghost.org/) blogging system, unlike Wordpress, does not support commenting system "out of the box". Native commenting support is not even on their roadmap of ghost development. But you can use Disqus, Google or Facebook comments on your ghost blog to add comment support.

We will be adding Disqus comments on our ghost blog.

To add Disqus comments, signup at [Disqus](https://disqus.com/. You will be asked for a sitename. Enter your desired sitename. After this, you will be provided with a html/javascript snippet like this:

{% highlight html linenos %}
<div id="disqus_thread"></div>
<script type="text/javascript">
 var disqus_shortname = 'yoursite';
 (function() {
   var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
   dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
   (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
 })();
</script>
<noscript>Please enable JavaScript to view the <a href="http://disqus.com/?ref_noscript">comments powered by Disqus.</a>
</noscript>
{% endhighlight %}

* Copy the code snippet.
* Now goto the directory where your ghost blog files reside. Assuming its name is ```ghostblog```, now browse to the folder ```ghostblog/content/themes/casper```.
* Here casper is the name of the default ghost theme that is used with ghost. If you are using any other theme, goto that theme's folder. Now there will be a file called ```post.hbs```.
* Open the file and somewhere after the ```</footer>``` tag, paste the copied code snippet. This will decide the location of comments on your page.
* Save the file and push the changes to your website host.
* You are done. Visit any post on your website to see the changes.
