---
layout: page
title: About
description: 畅游代码的世界
keywords: 逸梦灵尘, 逸梦, 灵尘, 周宇鹏, zhouyupeng
comments: true
menu: 关于
permalink: /about/
---

一双手敲一键盘

字符茫茫映眼前

缤纷世界自此始

且看数据化万千

## 联系

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
