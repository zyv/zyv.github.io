---
layout: default
title: Dr. rer. nat. Yury V. Zaytsev
---

Dr. rer. nat. Yury V. Zaytsev
=============================

<blockquote>
<p>Il n'y a pas de plus profonde solitude que celle de samouraï si ce n'est celle d'un tigre dans la jungle... peut-être...</p>

<p align="right">Le Bushidō</p>
</blockquote>

I was born and lived in USSR, majored in Radiophysics at [Nizhny Novgorod State University][nnsu] (Russia), obtained a doctorate in Computational Neuroscience from the [University of Freiburg][alu] (Germany), and have spent most of my conscious life pursuing a passion for _computing_. After a few years of rocking the boat at [Traveltainment GmbH][tt] (an [Amadeus][ama] company) as a senior software developer in data processing, I've joined [an exciting fintech startup][mm] as a software architect & technical lead.

[nnsu]: https://eng.unn.ru
[alu]: https://www.uni-freiburg.de
[tt]: https://www.traveltainment.de
[ama]: https://amadeus.com
[mm]: https://www.moneymeets.com

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

[1]: https://git-scm.com "Git rules!"
[2]: https://jekyllrb.com "Jekyll is cool, even though Ruby is not my cup of tea"
[3]: https://en.wikipedia.org/wiki/Human_brain "An organ that I should probably exercise more often"
