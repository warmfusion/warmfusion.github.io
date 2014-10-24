---
layout: page
permalink: /projects/
title: My Projects
tags: [hobbies]
modified: 2014-03-22
---

I have a number of different projects that I dabble with from time to time. Quite often these end up on GitHub

* [TheInternet](/TheInternet/)

See my github page for more


# Past Projects

- Ball monitor
    - This was envisioned as an "[Extreme feedback device](https://www.google.co.uk/search?q=extreme+feedback+device)" for Jenkins, email, weather etc
	- First attempts - [Controlling a single ORB](https://www.youtube.com/watch?v=8ruOxyTRj90)
	- [3 active orbs](https://www.youtube.com/watch?v=hlQ0-M2k6KE)
	- The project didn't get much further as I ran out of time/ability/enthusiasm
- [Jenk-O-Meter](https://github.com/warmfusion/jenk-o-meter)
	- A physical display of global build state for our [Jenkins CI](http://jenkins-ci.org/) build tool
	- Using lego, an arduino, a servo and a bit of PHP this display was mounted to a magnetic whiteboard in the office and the gauge drawn around it
	
# Future Ideas

## Clock Based Things
	
I've always found time keeping and in particular mechanisms themselves, incredibly facinating.

### Shelf Clock

The concept is to have a set of battery powered cubes placed around in any particular order which are managed from a central controller which orchestrates colour change events. They could be used for a number of visual feedback purposes, but I initially imagined a simple clock face made of 12 cubes placed across a bookshelf in roughly a circle where two colours are used - one representing hours (eg RED) and one for minutes (eg Gree) and the two colours fade and move around the clock following the time (Mixing accordingly when overlapping)

- A set of 12 illuminated battery powered cubes that communicate wirelessly to indicate the current time
- Making 12 wireless things is a pain
- Current plan to use MSP430s combined with some nrf2401 transceivers
- Likely use a Raspberry Pi as central controller as the additional power would allow for easier management of the light cluster

### EL Tape

Using some EL Wire Tape create a set of 7 segment display components and use them to create a large scale digital wall mounted clock.

Similar to the GPS clock described below, but by using el wire theres far less hassle with heat and light disipation and should result in a much thinner display.

Possible complications with power as EL Wire requires a more complex high voltage AC based supply, but this should be resolvable.

### EL Nixie Tube

Using EL wire create a Nixie Tube which doesn't need *quite* as much complicated setup. Maybe.

### Wheres Toby

Basically real world implementation of the Family clock from Harry Potter, which was originally planned to use Google Latitude but thats gone away so may need something else. Instead if may need to use iBeacons and get mobile phones to "ping home" when they're in certain places (eg work, car, home, etc). 

The current concept design uses a dismantled quartz clock where a continous rotation server or stepper motor will drive the minute and hour hands in tandem. This means that for any change in location a calculation needs to be made to effectivly point the two hands at (roughly) the right orientation.

For example, if 3=Lost, 6=Work,9=Travel, 12=Home, and two people are being "tracked" the clock times would need to be shown as follows:

| A(hr)  | B(min) | Time  |
|--------|--------|-------|
| Home   | Home   | 12:00 |
| Home   | Work   | 12:30 |
| Work   | Home   | 06:00 |
| Travel | Work   | 09:30 |


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
- Cross-Stitch desk ornamanent
	- Create something nice for my desk using cross stitch to convert some Pixel art into a physical object
	- ? As the material is quite thin, i wonder what a few white LEDs would look like behind key points - accenting elements such as eyes ?
- Herding Cats
        - A robot, or pair, which tries to locate an object or point by asking a user (or other robot) if hes cold, warm, hot or found it
        - A simple html page could present the four options and the robot then has to trianulate its position and where it suspects the object is


## Big Project Ideas

A few of the things I'd like to do require considerably more space or planning that my nebulous ideas above. These are highly unlikely to ever see the light of day, but are recorded for the one long weekend where enthusaism may take over and I need something big to occupy my time.


### UV Laser Phosphoresent Painter

We have a large blank wall in our office which I'd like to do something interesting with. One idea was to use sheets of phosphoresent paper to coat a large area of this space, then to use a computer controlled laser pointer to "draw" pictures which would fade over time. This would allow for the fact that this wall is high up (10 feet from the bottom) and would make for a display that could be changing over time.

The use of phosphoresent paper rather than simply leaving laser trails was considered as it would allow for longer or more detailed drawing as it would not be reliant on persistence of vision effects, and I believe it would be less distracting as the laster point would not be so harsh as it moves.

### Monster Clock

Basically replicate the [Spark Fun](http://www.sparkfun.com/) - [12 foot clock](https://www.sparkfun.com/tutorials/47)

