---
layout: page
title: CS 61A Fall 2016
category: cs61a
---

<div class="jumbotron">
# Thanks for a great semester, and good luck on the final!

**TA:** {{ site.owner }}
**Email:** [{{ site.email }}](<mailto:{{ site.email }}>)
(Please include "CS 61A" in the subject line.)

Check out [this page](http://jerryjrchen.com/cs61a/setup) if you're curious about my
development setup.


<a href="http://bit.do/jerrydisc" class="btn btn-raised btn-success">Submit
discussion attendance</a>
<a href="http://bit.do/jerrylabfb" class="btn btn-raised btn-warning">Lab
Feedback</a>
<a href="http://bit.do/jerrydiscfb" class="btn btn-raised btn-warning">Discussion
Feedback</a>
</div>

## Discussion Resources

| \# | Description                | Downloads |
|:---|:---------------------------|:----------|{% for r in site.data.cs61a.fa16.resources %}
| {{ r.number }} | {{ r.description }} | {% for d in r.downloads %} <a href="{{ d.link }}" class="btn btn-raised btn-default">{{ d.name }}</a>{% endfor %} {% endfor %}
{: .table .table-striped .table-hover #resources}

## Teaching Locations/Hours

| Type           | Time           | Location           |
|:---------------|:---------------|:-------------------|{% for loc in site.data.cs61a.fa16.locations %}
| {{ loc.type }} | {{ loc.time }} | {{ loc.location }} |{% endfor %}
{: .table .table-hover #resources}
