---
layout: post
title: Installing Oracle Solaris 10 on Sun Blade 1000
categories:
- tech
- solaris
---

Introduction
------------

So, you are a happy owner that wants to get latest and greatest Oracle Solaris running on a legendary `Sun Blade 1000` workstation with glorious double UltraSPARC III CPUs and a fair 1 Gb of RAM? Follow on!

Hardware requirements
---------------------

*   Check the version of OpenBoot firmware that your blade is equipped with. There is a known issue with OpenBoot 4.2 which prevents the workstations, having Toshiba DVD-ROM installed from booting from DVD media.

    This is a highly annoying problem. In general, two kinds of advices can be found on the Internets:

    - Find another DVD drive, replace the built-in Toshiba drive with a new one and hope that it will work. I have not tested this suggestion, but it might work for you.

    - Find the latest firmware upgrade, re-flash the firmware and re-install the OS. This solution will be described later on this page.

*   Be mindful of the fact, that graphical system requires at least 768 Mbs of RAM to be available. If this is not the case, you will only be able to use the text console. In my case, I had to open up the Blade and add more RAM from a donor machine that was missing a sound card.

Re-flashing the firmware
------------------------

Initially, I did not have an IDE DVD drive at hand, so I decided to go for the firmware upgrade, which proved to be more challenging that I would have ever expected. First, it took awhile to find out what is the internal Patch Number assigned to this particular firmware upgrade. For the record, it is `111292-17`. Second, it turned out, that apparently you need an active support contract with Oracle to be eligible for this download. This hindrance can be effectively worked around by Googling for `111292-17.zip`.

Now that the longed-for update is there, it is time to ask oneself how to proceed with the upgrade, given that the previously installed system is completely hosed (in this particular case, I have found some completely inoperable remnants of Solaris 8 scattered around the drive). The solution is easy: find a working Solaris LiveCD for SPARC, set up the networking, download the update from a private web server and proceed with the upgrade.

Investigation quickly revealed that [MilaX][mlx] is a perfect candidate for the job. Download the latest SPARC ISO (or Google for `milax032sparc.iso` if the website has already been taken down) and burn it on a *CD*.

Now, OpenBoot. One of the beauties of OpenBoot is that you can invoke the BIOS at any time on a running system (not that this might be the safest thing to do, though). So, in order to boot from the freshly burned image, press Stop+A on the Sun keyboard and type the following at the prompt:

    ok? setenv auto-boot? false

Reboot the machine and after inserting the CD and type the following to boot from the optical drive:

    ok? boot cdrom

The CD will spin up and after some time you will be presented with the login screen. Log into the system as user `alex` with password `alex`. Become root through `su -` with password `root`. Time to set the networking up and fetch the update:

```console
# ifconfig eri0 down
# ifconfig eri0 192.168.0.1 netmask 255.255.255.0 up
# route add default 192.168.0.254
# cat > /etc/resolv.conf
    nameserver 192.168.0.254
    ...
    Ctrl+D
# wget http://server.lan/flash-update-Blade1000-latest
# wget http://server.lan/flash-update-Blade1000-old
# wget http://server.lan/unix.flash-update.SunBlade1000.sh
# chmod +x unix.flash-update.SunBlade1000.sh
# ./unix.flash-update.SunBlade1000.sh
```

Answer `yes` and keep the fingers crossed; the magic will happen and the machine will reboot itself when the update is completed.

[mlx]: http://www.milax.org "MilaX, an OpenSolaris-based LiveCD"

Installing Solaris 10 proper
----------------------------

To be described. Refer to Blastwave. Provide some references, e.g. to Cuddletech.

Conclusion
----------

Overall, the firmware upgrade process went smoothly and is not nearly as scary as it sounds. However, few issues, such as the difficulty to find the patch and inability to use previously installed system to perform the upgrade made it necessary to do some prior research.

Solaris 10 installation went extremely smoothly and left feelings of mixed joy and sorrow. Even though I did the mistake of not pre-allocating space for the ZFS database and online upgrades, I probably will not need them anytime soon, so overall it seemed to be too easy to not have a catch.


