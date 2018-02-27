---
layout: post
title: Blogging with Jekyll - Legal and privacy
description: Legal and privacy aspects are important also to a plainionistic programmer.
tags: [jekyll, blogging]
series: "Blogging with Jekyll"
excerpt_separator: <!--more-->
---

Legal and privacy topics don't have primary focus on a blog but also such topics are important ...

<!--more-->

## Cookie Consent

In some countries every website has to give a clear warning to the user if it is using cookies.

The easiest way to achieve this is to 

1. Create a "_includes/cookieconsent.html" 
2. Go to [cookieconsent.insites.com](https://cookieconsent.insites.com/download/)
3. Configure, download and add your consent text to "_includes/cookieconsent.html" 
4. Include "_includes/cookieconsent.html" into your head.html

```
{% raw %}
{% include cookieconsent.html %}
{% endraw %}
```


## Robots.txt

Even though you write a blog to share content with your readers you may have pages you don't want to share with Google, e.g. imprint.
In order to hide such pages just create a robots.txt your Jekyll root:

```
---
---
User-agent: *
Disallow: /impressum
Disallow: /impressum/
Disallow: /impressum.html 
```

{% include series.html %}
