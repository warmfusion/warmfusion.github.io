---
layout: page
title:  "Performance Testing in CI"
tags: performance testing continous-integration stub
share: true
comments: true
---

Systems like Jenkins, Travis-CI, Bamboo and others make it so easy to automatically build, and functionally
test your code that I'm amazed at how little we leverage them for our non-functional testing.

Non-functional testing refers to anything in your application that isn't necessarily a "yes/no" behaviour; topics 
such as performance, usability, and security are considered non-functional as your product may well work but may 
take a week to respond, or make your users weep with frustration at poorly considered workflows.

A topic I'm interested in solving is around bringing what is typically a manual, after the fact action; "Great 
work on delivering that new feature, now lets see how it works in production" into our day to day build cycles.

I want to understand the ongoing behavioural changes in products by using a combination of automated
performance and security testing coupled with periodic human reviews so that when someone changes code under the hood
we know the impact it has made on the system. 

The problems that need to be solved, or at least embraced are:

* Reporting
    * How do we observe behaviour over time
* Monitoring
    * How do we know **what** caused the change in observed behaviour
* Consistency
    * Unlike functional testing, non-functional tests are more concerned with the environment
    upon which you are testing; how we do know the system is in a good steady state before running?
* Ease of use
    * Theres no point having a framework for these tests if its quickly made obsolete by developers
    failing to update tests as the product changes
    
# First Steps

As a competent developer 
I want to understand the performance of my application over time
So that I can make appropriate decisions about changing the product

So

1. Using a JMeter project performing simple API actions - as its something people tend to know and understand
2. Setup the performance-plugin in Jenkins
3. Bring the two together
4. Setup job for nightly execution
5. ???
6. Profit
