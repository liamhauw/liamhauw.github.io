---
title: Home
permalink: /
---

## Docs

{% assign docs_by_title = site.docs | sort: "title" %}

- Software and its engineering
  - Software notations and tools
    - General programming languages{% for doc in docs_by_title %}{% if doc.relative_path contains '/general_programming_languages/' %}
      - [{{ doc.title }}]({{ doc.url | relative_url }}){% endif %}{% endfor %}
- Theory of computation
  - Design and analysis of algorithms
    - Algorithm design techniques{% for doc in docs_by_title %}{% if doc.relative_path contains '/algorithm_design_techniques/' %}
      - [{{ doc.title }}]({{ doc.url | relative_url }}){% endif %}{% endfor %}
- Computing methodologies
  - Artificial intelligence
    - Natural language processing{% for doc in docs_by_title %}{% if doc.relative_path contains '/natural_language_processing/' %}
      - [{{ doc.title }}]({{ doc.url | relative_url }}){% endif %}{% endfor %}
    - Computer vision{% for doc in docs_by_title %}{% if doc.relative_path contains '/computer_vision/' %}
      - [{{ doc.title }}]({{ doc.url | relative_url }}){% endif %}{% endfor %}
  - Machine learning
    - Machine learning approaches{% for doc in docs_by_title %}{% if doc.relative_path contains '/machine_learning_approaches/' %}
      - [{{ doc.title }}]({{ doc.url | relative_url }}){% endif %}{% endfor %}
  - Computer graphics
    - Animation{% for doc in docs_by_title %}{% if doc.relative_path contains '/animation/' %}
      - [{{ doc.title }}]({{ doc.url | relative_url }}){% endif %}{% endfor %}
    - Rendering{% for doc in docs_by_title %}{% if doc.relative_path contains '/rendering/' %}
      - [{{ doc.title }}]({{ doc.url | relative_url }}){% endif %}{% endfor %}
    - Graphics systems and interfaces{% for doc in docs_by_title %}{% if doc.relative_path contains '/graphics_systems_and_interfaces/' %}
      - [{{ doc.title }}]({{ doc.url | relative_url }}){% endif %}{% endfor %}

### Reference

- [ACM computing classification system](https://dl.acm.org/ccs)
- [Stanford computer science program](https://bulletin.stanford.edu/programs?page=1&pq=Computer+Science)
- [Self-learning guide](https://csdiy.wiki/en/)

## Projects

{% assign projects_by_title = site.projects | sort: "title" %}
{% for project in projects_by_title %}
- [{{ project.title }}]({{ project.url | relative_url }}){% endfor %}

## Posts

{% for post in site.posts %}
- {{ post.date | date: "%Y-%m-%d" }} · [{{ post.title }}]({{ post.url | relative_url }}){% endfor %}
