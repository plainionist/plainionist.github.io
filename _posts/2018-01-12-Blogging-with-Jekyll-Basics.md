---
layout: post
title: Blogging with Jekyll - Basics
tags: [jekyll blogging]
series: "Blogging with Jekyll"
excerpt_separator: <!--more-->
---

How did I set up the basics of my blog?

Of course there are endless very good posts out there about Markdown + Jekyll + GitHub Pages and I am not going to repeat what others already have 
nicely documented. So let me just summarize my setup and referring to others for more detailed explainations.

<!--more-->

## Theme

I wanted a simple and "content focused" theme with a sidebar where I could refer to other resources.
I finally decided for [Lanyon](http://lanyon.getpoole.com/) which is pretty much what I would consider as a "plainionistic" design :)


## Excerpts

I think having excerpts of the most recent posts on the start page of the blog provides a nice overview to the visitor.
So I changed the index.html from

```
{{ {{ post.content }} }}
```

to

```
{{ post.excerpt }}
```

## Older posts

I think a blog should provide an easy access to the 10 to 15 most recent posts. This gives any easy overview for the visitors
what this blog is basically about. So I added the following to the sidebar:

```
<div class="sidebar-nav-item">
    <label>Older posts</label><br />
    {% for post in site.posts limit:10 %}
    <a href="{{ post.url }}">{{ post.title }}</a><br />
    {% endfor %}
</div>
```

## Discussions

And what is a blog without the option to leave comments?
[Disqus](https://disqus.com/) seems to be the state of the art solution so I created an include

```
{% if site.disqus %}
<div class="comments">
	<div id="disqus_thread"></div>
	<script type="text/javascript">

	    var disqus_shortname = '{{ site.disqus }}';

	    (function() {
	        var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
	        dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
	        (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
	    })();

	</script>
	<noscript>Please enable JavaScript to view the <a href="http://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
</div>
{% endif %}
```

And added it to the post layout

```
{% include disqus.html %}
```

That's it! My basic - plainionistic - setup is done ;-)

{% include series.html %}

