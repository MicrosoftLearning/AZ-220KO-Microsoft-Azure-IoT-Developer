---
title: 온라인 호스팅 지침
permalink: index.html
layout: home
---

# 콘텐츠 디렉터리

아래에 각 랩 연습과 데모의 하이퍼링크 목록이 나와 있습니다.

## 랩

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| 모듈 | 랩 |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

{% comment %}
<!-- 자리 표시자 데모가 나열되는 Jekyll 템플릿 주석 처리 -->

## 데모

{% assign demos = site.pages | where_exp:"page", "page.url contains '/Instructions/Demos'" %}
| 모듈 | 데모 |
| --- | --- | 
{% for activity in demos  %}| {{ activity.demo.module }} | [{{ activity.demo.title }}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

{% endcomment %}
