---
layout: page
title: About
description: 爱上你的命运
keywords: Ripper, ripperhe
comments: true
menu: 关于
permalink: /about/
---

> 爱上你的命运「Amor fati」

emmm，我是一个 iOS 开发，然后...

## Contact Me

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
