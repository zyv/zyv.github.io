---
layout: post
title: Tunneling TCP over UDP (DNS in particular)
categories:
- tech
---

Introduction
------------

This short post is just meant to be a recap on what I learned during my own very personal TCP over UDP tunneling quest.

First off, be mindful of the limitations of this technique. I have seen folks thinking that they are going to bypass TCP traffic shaping by tunneling it over UDP, since sometimes ISPs don't implement explicit policing for UDP, at least for known protocols, such as DNS, because, frankly speaking, from the ISP's POV it doesn't really make much sense. More precisely, the flood of UDP packets will still reach their network, no matter whether they decide to queue and drop them to make you suffer or not.

This simply doesn't work! UDP is mostly designed for streaming-like usage, i.e. you are not getting acknowledgements on received packets and server just goes on sending. Of course bi-directional communication is still possible in this scenario, but think of how much the performance of the applications that are designed with TCP in mind is going to degrade! It's just not worth it. Don't hurt your data! And if you are still thinking about re-implementing a better TCP on top of UDP (there must be no other reason why people keep polishing their TCP stacks for decades, other than that they are more stupid than you are), remember about the great demise of µTorrent `<g>` and think again.

Having that said, there might be valid reasons (of course, this depends on whether you include marginally unlawful activity in your definition of "valid" or not) to tunnel TCP over UDP. Such as, for instance, bypassing overly restrictive firewalls when you are set out to leak sensitive information to the outside world (an obligatory nonsensical example, since this goal can be achieved in hundreds of easier and safer ways).

Tunneling TCP streams over DNS
------------------------------

Now it is important to realise, that the fact that you are tunneling your traffic over UDP itself is of no help. You need to tunnel over something, that is not explicitly meant to be a bi-directional communication channel and that does not involve direct communication with the terminator of your tunnel.

For instance, there've been strange ideas to establish TCP over ICMP tunnels, but those are much more suitable to use as covert channels, since no sane sysop will leave such a blatant hole in his network. This is where DNS comes in. The great thing about DNS is that it is recursive by nature, which means that one can force a compliant DNS server to ask a very specific one (authoritative for the domain in question) to resolve a hostname if it doesn't know how to do it. Also, it's very commonly used and mostly considered harmless, which is also to our advantage.

That's why there are so many tools for this particular purpose and they mostly work even now that many sysops are starting to recognise that security is not something that one should keep taking lightly on the networks where sensitive information is transmitted.

Overview of the available tools
-------------------------------

* All this madness started with the legendary native implementation called [NSTX][nstx] by Tamas Szerb, which, however, doesn't seem to be under active development anymore and even hardly works.

* Later a Java re-implementation called [DNSCat][dnscat-old] by Tadek Pietraszek appeared. As the name suggests it's more like netcat in spirit. Based upon CNAME requests, which is painfully slow, but less prone to blocking.

* Dan Kaminsky (yes, the same guy which once succeeded in adding some randomness to the name resolution `<g>`) came up with a hacky ssh ProxyCommand compliant Perl script called [OzymanDNS][ozyman].

* Another native effort is called [iodine][iodine-project] and seems to be pretty active, lead by two Swedish guys, Bjorn Andersson and Erik Ekman.

* Yet another Java re-implementation exists by Tim Valenzuela of [tcp-over-dns][tcp-over-dns-project] fame. This is the one I've settled with. Works with TXT records by default, hopefully the author will implement CNAME support as well.

* There is an actively supported native cross-platform implementation called [dnscat][dnscat-new] by Ron Bowes.

[nstx]: http://savannah.nongnu.org/projects/nstx/ "NSTX at Savannah"
[dnscat-old]: http://tadek.pietraszek.org/projects/DNScat/ "The original DNSCat"
[ozyman]: http://www.doxpara.com "OzymanDNS by Dan Kaminsky"
[iodine-project]: http://code.kryo.se/iodine/ "The iodine project"
[tcp-over-dns-project]: http://analogbit.com/software/tcp-over-dns "The tcp-over-dns project"
[dnscat-new]: http://www.skullsecurity.org/wiki/index.php/Dnscat "The new dnscat"

Miscellaneous hints
-------------------

Here are the assorted things that I've learned over the past couple of days:

* Absolutely make sure that your DNS set up is correct (if you try to take a shortcut and put the IP address of your end point in the NS record directly the magic won't happen):

        tunnel.domain.tld.	IN	NS	ns.domain.tld.
        ns.domain.tld.	IN	A	123.123.123.123

* If you want to map an arbitrary port (think ssh) through dnscat, do it this way:

        mknod backpipe p
        nc 127.0.0.1 22 <backpipe | java -cp ... net.ibao.dnscat.DNScatServer -o tunnel.domain.tld -p 9876 1>backpipe

* The bash `while` syntax to make a resilient service is as follows:

        #!/bin/bash

        while :
        do
            /path/to/server
        done

* If you are running Ubuntu + ufw just put your REDIRECT rule in `/etc/rc.local`, the rest goes to `/etc/ufw/before.rules`. Don't forget to enable forwarding in `/etc/ufw/sysctl.conf` (reboot to apply).

* The correct dig syntax is as follows (just FYI, I always keep forgetting it):

        dig any sub.domain.tld @ns.server.tld

* I always forget the correct nmap syntax for host fingerprinting and keep on googling:

        sudo nmap -A host.tld
        sudo nmap -O host.tld

* Nice ~/.ssh/config to use with tcp-over-dns (commented part is for DNSCat)... Enjoy a SOCKS5 proxy server on localhost:8888.

        Host tunnel
          HostName localhost
          Compression yes
          ForwardX11 yes
          IdentityFile ~/.ssh/id_dsa
          #Port 22
          Port 9876
          ServerAliveInterval 30
          TCPKeepAlive yes
          User name
          #ProxyCommand $HOME/bin/dnscat-0.02/DNScatClient -o tunnel.domain.tld
          StrictHostKeyChecking no
          UserKnownHostsFile /dev/null
          DynamicForward 8888

What is left to be done is probably to find a nice and easy to use socksifier for Linux, so that one, for instance, can do something along the lines of:

    socksify git fetch

and enjoy happy coding while on train.

Thanks to everyone involved!

