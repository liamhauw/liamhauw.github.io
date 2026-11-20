---
title: Home
permalink: /
---

## Docs
The categories below are based on the [ACM/IEEE-CS/AAAI computer science curricula](https://csed.acm.org).

{% assign docs_by_title = site.docs | sort: "title" %}

- Artificial intelligence{% for doc in docs_by_title %}{% if doc.relative_path contains '/ai/' %}
  - [{{ doc.title }}]({{ doc.url | relative_url }}){% endif %}{% endfor %}

- Algorithmic foundations{% for doc in docs_by_title %}{% if doc.relative_path contains '/al/' %}
  - [{{ doc.title }}]({{ doc.url | relative_url }}){% endif %}{% endfor %}

- Graphics and interactive techniques{% for doc in docs_by_title %}{% if doc.relative_path contains '/git/' %}
  - [{{ doc.title }}]({{ doc.url | relative_url }}){% endif %}{% endfor %}

- Software development fundamentals{% for doc in docs_by_title %}{% if doc.relative_path contains '/sdf/' %}
  - [{{ doc.title }}]({{ doc.url | relative_url }}){% endif %}{% endfor %}

- Specialized platform development{% for doc in docs_by_title %}{% if doc.relative_path contains '/spd/' %}
  - [{{ doc.title }}]({{ doc.url | relative_url }}){% endif %}{% endfor %}

## Projects

{% assign projects_by_title = site.projects | sort: "title" %}
{% for project in projects_by_title %}
- [{{ project.title }}]({{ project.url | relative_url }}){% endfor %}

## Posts

{% for post in site.posts %}
- {{ post.date | date: "%Y-%m-%d" }} · [{{ post.title }}]({{ post.url | relative_url }}){% endfor %}
