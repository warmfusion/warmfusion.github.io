---
layout: post
title:  "Building The Internet"
categories: msp430 hobbies 
share: true
comments: true
---

# Built yourself The Internet

![The Internet](/images/the_internet.jpg)

A really simple weekend project using some serious overengineering to create a black box with a blinky light.

List of components;

* 2x 1.1K resistors
* MSP430g2231 Microcontroller
* 1 Red LED (5mm)
* AA Battery holder  
* Stripboard
* Small wires(for jumpers)
* Black Hobby Box

I’d have liked a nice LED mount, but Maplin didn’t have any available, so that might be a later improvement.

The Idea - Make a blinky box that matches the “feel” of The Internet from the [IT Crowd](http://www.youtube.com/watch?v=iDbyYGrswtg)

I wanted to create a properly wireless device, with as long life as possible. Originally planned to have a few solar cells for indefinite runtime, but I couldn’t find any in my “Spare parts” drawer. Instead, after failing to work out how proper electrical engineers do blinky things, settled on using a microcontroller. 

Aside from being horribly overengineered, I can excuse myself by suggesting that it may be extended at later dates with some features like an RGB LED, LDRs etc. For now… *shrugs*

Again, I needed it to be long lasting; I don’t want to waste batteries on such a thing, so I spent some time looking at running the MSP430 in low power mode, along with some interrupt coding.

The code is available from my GitHub repository here: (warmfusion/TheInternet)[https://github.com/warmfusion/TheInternet]

The basics are;

* Set the P1 ports to OUTPUT as this reduces current consumption on the MSP430 as it doesn’t try and read any input 
* Turn on the input VLO oscillator ready to start counting
* Ensure interrupts are going, and go into Low Power Mode 3 (LPM3)
* After a delay, the interrupt is called, toggling the LED 
* As an addition, I change the cycle counter on each toggle so that the LED blink has a longer pause than the on time.


