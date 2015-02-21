---
layout: page
title:  "Event Centralisation not Log Centralisation"
tags: musing
share: true
comments: true
---

Logging has become a hot topic in many DevOps circles, with tools like [Graylog](https://www.graylog.org/), [LogStash](http://logstash.net/), and [Splunk](http://www.splunk.com/) becoming widly known and used by many teams to automatically filter, parse and store logs in a central database.

However, I believe the community has yet been able to take these tools to their fullest potential as the focus has been on *log* centralisation, and not *event* centralisation.

## Log vs Event

What do I mean by 'event', and why is it so different from a 'log'?

Logs are records of actions, with no due consideration to their relevance to workflows, users, or even the log consumers themselves. Logging is often created by developers who are debugging their application, and simply write out information as they saw fit at the time. Obviously main-stream applications like webservers, message brokers, and databases standardise their own logs to some extend, and people create filters to interpret those logs, but ultimatly they're still arbitrary strings.

Events however, are standardised logging messages, created to indicate a particular business relevant action has occurred. Ideally, event messages contain unique identifiers, timestamps, and enough meta-data to allow end-users to create a general purpose log parser.

### Show me...

Lets take a typical logging example from an nginx server

    192.33.10.1 - - ['01/Dec/2012:12:34:56 +0000'] "POST /some/secure/path HTTP/1.1" 403 6642 "-" "Mozilla/5.0 (Windows NT 6.3; WOW64; rv:34.0) Gecko/20100101 Firefox/32.0"

This tells me that someone made a request for a uri, that failed some sort of authentication, resulting in our service returning a forbidden response. But who, and what were they trying, did it fail authentication, or authorisation.

But this is what an event may look like, formatted for ease of reading:

{% highlight json %}

    { 
      "metadata": { 
        "parent_id": "53f7ff213b3f31336a2e323da58d984d", 
        "id": "3e7ef673c408256a1f29a3ebd23bfe23",
        "type": "access",
        "tags": [ "security", "unauthorised" ],
        "timestamp": "01/Dec/2012:12:34:56 +0000"
      },
      "event": {
        "message": "Unauthorised access attempt"
        "detail": "User X attempted to update User Y profile"
        "user": "user-x-id",
        "severity": "WARNING",
      }
    }

{% endhighlight %}



