---
layout: base_layout
title: Courses
metaTitle: All Customer Service Training Courses | Dublin Cork Limerick Ireland
description: Every Customer Service training course we run, with outlines and dates. Instructor-led courses in Dublin, Cork, Limerick and live online.
tags: [main]
pageType: courses
---
<header class="page-head">
<div class="wrap">
<h1>{% for keyword in site.searchKeywords %}{{ keyword }} {% endfor %}training courses</h1>
<p class="page-head__lead">Instructor-led courses in Dublin, Cork, Limerick and live online. Choose a course to see what it covers and when it runs. <a href="#contact">Contact us</a> with any queries.</p>
</div>
</header>

<div class="wrap page">
<ul class="course-list">
{% assign coursesUpcomingFirst = coursesapifull.courses | upcomingFirst %}
{% for course in coursesUpcomingFirst %}
<li>
<h2><a href="/courses/{{ course.name | slug | removeInvalidChars }}/">{{ course.name }}</a></h2>
<p>{{ course.descrip | shorten: 220 }}</p>
{% assign meta = course | courseMeta %}{% if meta != "" %}<p class="course-list__meta">{{ meta }}</p>{% endif %}
</li>
{% endfor %}
</ul>
</div>
