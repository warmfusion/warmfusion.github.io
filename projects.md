---
layout: page
permalink: /projects/
title: Projects
description: "A list of the various projects I've created"
---

Here is a list of projects I'm working on in my spare time, they're in a variety of stages and they often involve clocks in some way, but take a look if you're interested.

<ul class="post-list">
{% for project in site.projects %}
  <li><article><a href="{{ site.url }}{{ project.url }}">{{ project.title }}
  {% if project.icons %}
  <span style='float:right'>{% for icon in project.icons %} <i class="icon-{{ icon }}"></i>  {% endfor %}</span>
  {% endif %}
  <footer>{{ project.summary }}</footer></a></article> </li>
{% endfor %}
</ul>



## Nebulous Ideas - aka The Weekender

These are project ideas which havn't quite gotten off the ground in any substantial way. They're probably weekend jobs, or perhaps even just a few hours, but I always need just one more part, or my desktop isn't quite running the right software so they've not gotten much further than being ideas.

- Tea Counter
    - Using the load cell from a set of digital scales, measure the weight of a kettle and guesstimate how many brews are consumed over time
- Sandwich Safe
    - Add some pointless blinky LEDs, NFC displays and other gubbins to my sandwich box (which looks like a flight case)
- Strain Gauge
	- Using data of active transactions per minute, create a physical representation of the data using a old fashioned steam gauge
- Digital Picture Frame
	- Create a digital picture frame which shows pictures retrieved from Facebook/Twitter friends so its always kept up to date
- Herding Cats
        - A robot, or pair, which tries to locate an object or point by asking a user (or other robot) if hes cold, warm, hot or found it
        - A simple html page could present the four options and the robot then has to trianulate its position and where it suspects the object is
