---
layout: post
title: Example entry 1
category: bpexample
author: Max Mustermann
tags:
- example
- jekyll
---

## Include Images

Just use this:

{% raw %}
```
{% include image.html url="/bpexample/images/Example.jpg" description='Resulting image with caption and embedded html <a href="https://google.com">link</a>' %}
```
{% endraw %}


{% include image.html url="/bpexample/images/Example.jpg" description='Resulting image with caption and embedded html <a href="https://google.com">link</a>' %}

<!--more-->

## Syntax highlighting

Use the highlight tags:

{% raw  %}
```
{% highlight python %}
#comment
def get_cool_number(n):
  x = n
  x = 42
  return 42
{% endhighlight %}
```
{% endraw  %}

{% highlight python %}
#comment
def get_cool_number(n):
  x = n
  x = 42
  return 42
{% endhighlight %}

## Markdown

Markdown is supported, for further documentation refer to the [jekyll docs](https://jekyllrb.com/docs/posts/).