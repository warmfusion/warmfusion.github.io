---
layout: page
title:  "IPv6 on my network"
tags: technical network
share: true
comments: true
---

# Whats IPv6?

IPv6 is a 'new' method of identifying devices uniquely on the internet which amongst other useful features
allows for a considerably larger number of machines to be connected at once. You can find plenty of information
about IPv6 on the internet, so rather than trying to repeat other peoples work I'll just explain my motivations
for my experimentation and assume you have some moderate level of technical ability.

1. Learning experience
    * It's inevitable that IPv6 will become used more and more frequently in my professional work and
      if I want to remain relevant it makes sense to experiment with this technology sooner rather than
      later
2. Internet of Things (IoT)
    * As I create or purchase more and more network enabled devices I'm using IP addresses at home but
    I can't make them publically accessible - IPv6 lets me trivially access these devices publically
3. Docker
    * Like (1) combined with (2), it lets me explore allocating IPv6 addresses to docker containers and
    setup more complex network stacks without having to use subnet tricks

# Equipment

I'm using an [Asus RT-N16](www.dd-wrt.com/wiki/index.php/Asus_RT-N16) flashed with the DD-WRT firmware connected
to a Virgin Media SuperHub set in Modem mode. This means all real network routing work is handled by the Asus
device, and I can set it up to manage an IPv6 network transparently across my whole LAN (including WiFi) for 
clients that can support it.


# The Process

## Step 1 - Obtain an IPv6 address

I'm using the (Highly recommended) [Hurricane Electric TunnelBroker](https://tunnelbroker.net/) service which
provides a free IPv6 /64 range. It can be a little bit technical to setup, but if you're reading this far I'm
going to assume some level of technical ability.

> If you select a broker location near home you should get excellent performance; but if you're feeling adventurous
> it _may_ be possible to select an international broker endpoint and use your IPv6 traffic as a rudementary proxy

## Step 2 - Enable IPv6

Navigate to `Administration >> Management` and enable IPv6 and Radvd.

In the Radvd configuration section, add the following config block and ensure you update the
`<REMOTEIP>` with your IPv6 prefix that was just created. For example, `2001:db8:0::/64` 


    interface br0 {
      MinRtrAdvInterval 3;
      MaxRtrAdvInterval 10;
      AdvLinkMTU 1480;
      AdvSendAdvert on;
      prefix <REMOTEIP>::/64 {
        AdvOnLink on;
        AdvAutonomous on;
        AdvValidLifetime 86400;
        AdvPreferredLifetime 86400;
        # Base6to4Interface vlan2;
      };
    };

## Step 3 - Install ip6tables

> This step is incomplete - I've not worked out how to set this up, and that basically means
> all my devices are currently exposed to the world-wild-web.

If you want to protect all your IPv6 devices, which will be fully exposed to the internet when they get an IPv6 
address, you will either need to run firewalls (or equivalent) on each of the clients, or install `ip6tables` on
your router itself (Or get a dedicated network firewall).

Once again the DD-WRT community come to the rescue with instructions on [how to install ip6tables on dd-wrt](http://www.dd-wrt.com/wiki/index.php/IPV6#ip6tables_for_K26_big_images) where the basic idea is:

1. Download or compile the modules
2. SCP the .ko files to `/jffs/lib/modules/2.6.24.111`
    * This assumes you've setup the `Secure Shell` access to your router
3. ... TBC


## Step 3.1 - Userland things

The ASUS RT-N16 uses a Broadcom chip, so you could download pre-compiled modules from [here](http://downloads.openwrt.org/kamikaze/8.09.2/brcm47xx/packages)  

## Step 4 - Script the IPv6 configuration

Luckily, the DD-WRT community were able to help set this up, with a very simple [tutorial](http://www.dd-wrt.com/wiki/index.php/IPv6#Hurricane_Electric.27s_Tunnelbroker.net)

Using the tutorial above, and some tinkering, I was able to get my IPv6 configuration working using the following
simple set of commands:

    SERVER_IP4=<TunnelBroker Server IPv4 Address>
    CLIENT_IP4=<Your Public IPv4>

    REMOTE_IP6=<TunnelBroker Server IPv6 Address> # Eg 2001:db8:0:1/64
    CLIENT_IP6=<TunnelBroker Client IPv6 Address> # Eg 2001:db8:0:2/64


    #Ensure IPv6 modules are loaded
    insmod ipv6
    # Create a IPv6 over IPv4 tunnel
    ip tunnel add he-ipv6 mode sit remote ${SERVER_IP4} local ${CLIENT_IP4} ttl 255
    ip link set he-ipv6 up

    # Set our Remote IPv6 address on the tunnel device
    ip addr add ${REMOTE_IP6} dev he-ipv6

    # Transmit all IPv6 traffic through this tunnel
    ip route add ::/0 dev he-ipv6

    # Setup IPv6 on the Routers internal Bridge device
    ip -6 addr add ${CLIENT_IP6} dev br0


    # Truth be told - I have no idea why this is needed :-(
    ROUTED_ADDRESS=`sed -n -e 's,^ *prefix *\([^ ]*\) *{,\1,p' /tmp/radvd.conf`
    BR0_MAC=$(ifconfig br0 |sed -n -e 's,.*HWaddr \(..\):\(..\):\(..\):\(..\):\(..\):\(..\).*,\1\2:\3\4:\5\6,p')
    ip -6 addr add $(echo "$ROUTED_ADDRESS"|sed "s,::/..,::$BR0_MAC/64,") dev br0
    ip -6 route add 2000::/3 dev he-ipv6

    # Let the router boot up, otherwise radvd doesn't work
    sleep 5

    # Start the Router Advertisement Daemon so machines on the LAN get some IPv6 loveliness
    radvd -C /tmp/radvd.conf

## Step 5 - Persistence

Goto `Administration >> Commands` and copy/paste the script above, changing the configuration variables
as required, and save as `startup`.

Reboot the router, and everything _should_ just work.



