---
layout: page
title: Find-A-Tobe
summary: Device for showing Tobys rough location via a clock-face
permalink: /projects/find-a-tobe/
icons: 
  - time
  - map-marker
---

Basically real world implementation of the Family clock from Harry Potter, which was originally planned to use Google Latitude but thats gone away so may need something else. Instead if may need to use iBeacons and get mobile phones to "ping home" when they're in certain places (eg work, car, home, etc). 

The current concept design uses a dismantled quartz clock where a continous rotation server or stepper motor will drive the minute and hour hands in tandem. This means that for any change in location a calculation needs to be made to effectivly point the two hands at (roughly) the right orientation.

For example, if 3=Lost, 6=Work,9=Travel, 12=Home, and two people are being "tracked" the clock times would need to be shown as follows:

| A(hr)  | B(min) | Time  |
|--------|--------|-------|
| Home   | Home   | 12:00 |
| Home   | Work   | 12:30 |
| Work   | Home   | 06:00 |
| Travel | Work   | 09:30 |