---
layout: default
title: Dr. rer. nat. Yury V. Zaytsev
---

Dr. rer. nat. Yury V. Zaytsev
=============================

<blockquote>
<p>Il n'y a pas de plus profonde solitude que celle de samouraï si ce n'est celle d'un tigre dans la jungle... peut-être...</p>

<p align="right">Le Bushido</p>
</blockquote>

I was born in USSR, majored in radiophysics, am now working on a doctorate in computational neurosciences and have spent most of my conscious life doing computer science.

The views expressed on this page are entirely of my own and do not reflect the official position of my employer, relatives, acquaintances et cetera.

<h2 class="feed-link"><a href="/atom.xml" title="Atom Feed" class="feed-href">Recent press releases</a></h2>

### Geekery and technicalities

<ul class="posts-list">
    {% for post in site.categories.tech %}
        <li><a href="{{ post.url }}">{{ post.title }}</a> <span>({{ post.date | date_to_long_string }})</span></li>
    {% endfor %}
</ul>

### Miscellanea

<ul class="posts-list">
    {% for post in site.categories.misc %}
        <li><a href="{{ post.url }}">{{ post.title }}</a> <span>({{ post.date | date_to_long_string }})</span></li>
    {% endfor %}
</ul>

Credits
-------

I would like to acknowledge the use of the following entities, being instrumental in creation of this webpage:

- [Git][1]
- [Jekyll][2]
- [Brain][3]

[1]: http://git-scm.com "Git rules!"
[2]: http://jekyllrb.com "Jekyll is cool, even though Ruby is not my cup of tea"
[3]: http://en.wikipedia.org/wiki/Human_brain "Something that I should probably exercise more often"
