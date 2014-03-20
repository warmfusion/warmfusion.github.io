---
layout: post
title:  "Setting up Shinken on OMD"
categories: monitoring shinken nagios
---

# Whats Shinken
http://www.shinken-monitoring.org/

# How do I get it?

I'm using OMD here, which means its pretty simple to configure Shinken as an alternative check core to Nagios.

## Known Issues

A bug in OMD that stops you from adding new hosts and applying changes from the check_mk UI. It appears that OMD thinks nagios.cfg exists even 
when using Shinken and so when applying those changes the system falls over. The workaround is to provide a nagios.cfg from the shinken configuration,
as well as configure the check_submission system (I dont understand that bit though).

    ln -s /omd/sites/<yoursite>/tmp/shinken/shinken-apache.cfg /omd/sites/<yoursite>/tmp/nagios/nagios.cfg
    echo "check_submission = 'pipe'" >> /omd/sites/<yoursite>/etc/check_mk/main.mk

## Swap Nagios out

Lets make a new site called "shishi" and swap out Nagios for Shinken

    omd create shishi
    omd config shishi set CORE shinken

## Add Contact Group

To allow users to login to the Shinken Web UI, you need to configure a contact configuration for Nagios (odd but true)
Enter the following block of configuration into /omd/sites/<mysite>/etc/nagios/conf.d/contact.cfg and restart your omd site

    define contact {
      contact_name                  omdadmin
      alias                         omdadmin admin contact
      host_notification_commands    check-mk-dummy
      service_notification_commands   check-mk-dummy
      host_notification_options     n
      service_notification_options  n
      host_notification_period      24X7
      service_notification_period   24X7
    }
    
    define contactgroup {
      contactgroup_name             omdadmin
      alias                         omdadmin admin contact group
      members                       omdadmin
    }
 
## Enable MongoDB for Dashboard persistence

Next enable mongodb so Shinken can store its dashboard configuration. The port may need to be changed

    omd config shishi set MONGODB on
    omd config shishi set MONGODB-TCP_PORT 27018
    
Add some configuration to mongodb to shinkens module_webui.cfg - Lines 13 , 22-27


    define module{
        module_name      WebUI
        module_type      webui
        host             0.0.0.0       ; mean all interfaces
        port             7767
        # CHANGE THIS VALUE!!!!!!!
        auth_secret      CHANGE_ME
        # Advanced options. Do not touch it if you don't
        # know what you are doing
        #http_backend    wsgiref
        # ; can be also : cherrypy, paste, tornado, twisted
        # ; or gevent
        modules          Apache_passwd,Mongodb
        # Modules for the WebUI.
    }
    define module{
        module_name      Apache_passwd
        module_type      passwd_webui
        # WARNING : put the full PATH for this value!
        passwd           /omd/sites/shishi/etc/htpasswd
    }
    define module{
      module_name Mongodb
      module_type mongodb
      uri mongodb://localhost:27018
      database shinken
    }
   
# Next Steps

## Bring The Graphite

http://www.shinken-monitoring.org/wiki/use_with_graphite

## Reap the rewards

* Learn more about what makes Shinken different to Nagios by experience
* See if I can get better integration of Monitored events against continuous datasets
    * For example, I want to show when a Jenkins build occured on a build node as a vertical line on my CPU monitors for that node.

 
# References

Alot of googling around was needed to bring these pieces together, here are the main sources of my "knowledge"

* Quick start installation/configuration of Shinken - https://labs.consol.de/blog/nagios/how-to-install-bleeding-edge-shinken-in-a-minute-with-omd/
* Logging into Shinken (Contact.cfg) - http://www.monitoring-portal.org/wbb/index.php?page=Thread&postID=163198#post163198
* MongoDB Configuration example taken from - http://www.shinken-monitoring.org/forum/index.php?topic=605.0
