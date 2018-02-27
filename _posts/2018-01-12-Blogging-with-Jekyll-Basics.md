---
layout: post
title: Blogging with Jekyll - Basics
description: My basics to blog with Jekyll. Short, simple, plainionistic.
tags: [jekyll, blogging]
series: "Blogging with Jekyll"
excerpt_separator: <!--more-->
---

How did I set up the basics of my blog?

Of course there are endless very good posts out there about Markdown + Jekyll + GitHub Pages and I am not going to repeat what 
others already have nicely documented. So let me just summarize my setup and referring to others for more detailed explanations.

<!--more-->

## Theme

I wanted a simple and "content focused" theme with a sidebar where I could refer to other resources.
I finally decided for [Lanyon](http://lanyon.getpoole.com/) which is pretty much what I would consider as a "plainionistic" design :)


## Excerpts

I think having excerpts of the most recent posts on the start page of the blog provides a nice overview to the visitor.
So I changed the index.html from

```
{% raw %}
{{ post.content }}
{% endraw %}
```

to

```
{% raw %}
{{ post.excerpt }}
{% endraw %}
```

## Discussions

And what is a blog without the option to leave comments?
[Disqus](https://disqus.com/) seems to be the state of the art solution so I created an include

```
{% raw %}
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
{% endraw %}
```

And added it to the post layout

```
{% raw %}
{% include disqus.html %}
{% endraw %}
```

That's it! My basic - plainionistic - setup is done ;-)

{% include series.html %}

