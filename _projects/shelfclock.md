---
layout: page
title: Shelf Clock
summary: Twelve coloured boxes showing time through colour
permalink: /projects/shelfclock/
icons: 
  - clock-o
---

# Shelf Clock

The concept is to have a set of battery powered cubes placed around in any particular order which are managed from a central controller which orchestrates colour change events. They could be used for a number of visual feedback purposes, but I initially imagined a simple clock face made of 12 cubes placed across a bookshelf in roughly a circle where two colours are used - one representing hours (eg RED) and one for minutes (eg Gree) and the two colours fade and move around the clock following the time (Mixing accordingly when overlapping)

- A set of 12 illuminated battery powered cubes that communicate wirelessly to indicate the current time
- Making 12 wireless things is a pain
- Current plan to use MSP430s combined with some nrf2401 transceivers
- Likely use a Raspberry Pi as central controller as the additional power would allow for easier management of the light cluster
