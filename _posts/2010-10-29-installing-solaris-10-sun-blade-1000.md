---
layout: post
title: Installing Oracle Solaris 10 on Sun Blade 1000
---

Introduction
------------

So, you are a happy owner that wants to get latest and greatest Oracle Solaris running on a legendary Sun Blade 1000 workstation with glorious double UltraSPARC III CPUs and a fair 1 Gb of RAM? Follow on!

Hardware requirements
---------------------

*   Check the version of OpenBoot firmware that your blade is equipped with. There is a known issue with OpenBoot 4.2 which prevents the workstations, having Toshiba DVD-ROM installed from booting from DVD media.

    This is a highly annoying problem. In general, two kinds of advices can be found on the Internets:

    - Find another DVD drive, replace the built-in Toshiba drive with a new one and hope that it will work. I have not tested this suggestion, but it might work for you.

    - Find the latest firmware upgrade, re-flash the firmware and re-install the OS.

* Graphics

Re-flashing the firmware
------------------------

Initially, I did not have a 

111292-17.zip


