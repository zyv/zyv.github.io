---
layout: default
title: Dipl. Phys. Yury V. Zaytsev
---

Dipl. Phys. Yury V. Zaytsev
===========================

I am working on a doctorate in computational neurosciences, graduated from a physics department and having spent most of my conscious life doing computer science.

The views expressed on this page are entirely of my own and do not reflect the official position of my employer, relatives, acquaintances et cetera.

Recent press releases
---------------------

<ul class="posts">
    {% for post in site.posts %}
        <li><span>{{ post.date | date_to_string }}</span> &ndash; <a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
</ul>

Credits
-------

I would like to acknowledge the use of the following entities, being instrumental in creating this page:

- [Git][1]
- [Jekyll][2]
- [Brain][3]

    [1]: http://git-scm.com
    [2]: http://jekyllrb.com
    [3]: http://en.wikipedia.org/wiki/Human_brain
