---
layout: page
title: Notepad
---
{% include JB/setup %}
###INDEX

![](/images/eye.png)
####Talking the talk is not as good as walking the walk

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

---
### LINK

- [Fork me on github](https://github.com/shonwang/shonwang.github.com)
- [Ji Wei's github](https://github.com/chengzi)
- [Ya's github](https://github.com/dracher)
- [Python自动单元测试框架](http://www.ibm.com/developerworks/cn/linux/l-pyunit/index.html)

---
### To-Do

Read [Jekyll Quick Start](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)

Complete usage and documentation available at: [Jekyll Bootstrap](http://jekyllbootstrap.com)

This theme is still unfinished. If you'd like to be added as a contributor, [please fork](http://github.com/plusjade/jekyll-bootstrap)!
We need to clean up the themes, make theme usage guides with theme-specific markup examples.


