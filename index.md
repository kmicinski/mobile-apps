---
layout: mainpage
---

## Upcoming deadlines

<ul class="due-list">
{% for post in site.posts %}
    {% capture nowunix %}{{'now' | date: '%s'}}{% endcapture %}
    {% capture duetime %}{{post.due | date: '%s'}}{% endcapture %}
    {% if post.categories contains 'assignment' and duetime > nowunix %}
    <li>
       <span><span class="post-meta"><b>(Due <span itemprop="date">{{ post.due | date: "%b %-d, %Y" }}</span>)</b></span><a class="mainpage-asn-link" href="{{ post.url | absolute_url }}">{{ post.title }}</a></span></li>
   {% endif %}
{% endfor %}
</ul>

## Information

<div class="infomatter">
<table class="infotablestyle">
<tr><td>Course Number</td>
    <td>CMSC 107 (Fall 2018) at <a href="https://www.haverford.edu/computer-science/">Haverford College</a></td>
</tr>
<tr><td>Instructor</td>
    <td><a href="http://kmicinski.com">Kristopher Micinski</a></td>
</tr>
<tr>
    <td>Times</td>
    <td>Tu/Thur 11:30-13:00 <i>Lecture</i>  W 11:30 / 12:30 <i>Labs</i></td>
</tr>
<tr>
    <td>Office Hours</td>
    <td>Tu/Thur after class and by appointment</td>
</tr>
</table>
<img class="krispic" src="/assets/img/krisbw.jpg">
</div>
    
## Introduction 

When I was learning to program, I had a deep anxiety about my
inability to articulate my thoughts in a way that felt natural to the
languages I was using. I was able to write small programs, but as they
grew in complexity, they also became more brittle and less robust. In
this course, our goal is to learn how to tackle that challenge: how to
scale up our thinking about small programs to building large, robust,
well-maintained systems.

We're going to do this by building mobile apps using the Android
operating system. Specifically, students will form groups to propose
and build an app to enable some form of social change. Some groups
will likely work with nonprofits, others will work on ideas of their
own creation. The common theme is that the apps we're building are
intended to improve the lives of others in some positive way.

Along the way, we're going to be focusing on essential concepts in
software engineering. We'll learn concepts like test-driven
development, encapsulation, information hiding, design patterns, and
more. And as we build larger and larger software, we'll constantly
reflect upon the code we're writing to ensure we're delivering a
product that's fit for the people for whom we're building it!

I hope you're excited for the course as I am!

## Course Structure

Please read the [Syllabus]({{ "/syllabus" | absolute_url }}) for course information.
