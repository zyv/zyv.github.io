---
layout: post
title: Who framed mkfs? or Winners never cheat and cheaters never win
categories:
- tech
---

I/O Limits: Quest for increased drive capacity
----------------------------------------------

My first Armstrad had a glorious 20M hard drive and now I sometimes feel dissatisfied about the capacity of a shiny new 2T RAID array. Some *1 000 000x* increase in 15 years, in fact, does not that sound quite amazing when one thinks about it?

The storage devices used to expose the data in 512B large addressable blocks to the OS. However, as their capacity has been steadily increasing, it became clear that the overhead associated with each sector on the current 512 byte sector disks is becoming a limiting factor.

In the reality, apart from 512 bytes worth of data, each physical sector contains quite a bit of extra information, such as an error correction checksum, sync label, etc. In some cases, these overheads amounted up to 15% of the usable storage space. For reasonably large drives of 2T and more we might be speaking about some 300G lost due to the layout format inefficiency.

Therefore, as storage vendors quickly realized, one way to obtain a major increase in capacity is to reduce the overheads associated with storing each physical sector on disk. That is why most of the modern hard drives operate with 4K sectors internally (`physical_block_size`), while exposing 512B sectors (`logical_block_size`) to legacy software.

However, this trickery soon enough led another annoying problem: because the software layer on top of the drive's firmware is still thinking that it operates with 512B sectors internally, it could easily happen that larger logical blocks in use by the file system would become mis-aligned against physical 4K sectors that the drive is dealing with internally.

In such a case each unaligned I/O operation requested by the OS would cause the drive to perform a Read-Modify-Write (RMW) highly impacting the performance by reducing IOPS and increasing the latency. RMWs can be to a certain extent mitigated in firmware, but the drive just does not have enough information about the real needs of the OS to completely eliminate the problem.

That is why it has been finally agreed, that instead of building error-prone and extremely complicated kludges into the firmware, it makes much more sense to expose the information about the preferred sector sizes and alignment to the OS and propagate it to the upper layers such that partitioning, file system creation tools, etc. would be aware of it.

One way or another, the Linux I/O stack (starting from Linux >= 2.6.31) has been enhanced to consume vendor-provided information about the I/O limits [ms-1] that allows Linux tools (parted, lvm, mkfs.\*, etc.) to optimize the placement of and access to the data (see also [ms-2] for other very interesting documents regarding this issue). Also, be sure to check out a very interesting article by Tejun Heo [ko] regarding the sector size issues in general (thanks to Slyfox for the link).

[ms-1]: http://people.redhat.com/msnitzer/docs/io-limits.txt "I/O Limits: block sizes, alignment and I/O hints"
[ms-2]: http://people.redhat.com/msnitzer/docs/ "Home page of Mike Snitzer, Red Hat"
[ko]: https://ata.wiki.kernel.org/index.php/ATA_4_KiB_sector_issues "ATA pages @ kernel.org wiki"

The buggy SSD firmware vs. minimum_io_size
------------------------------------------

Now, on the practical side, even though RHEL6.1, for instance, has a complete support for I/O limits, not all devices (especially the legacy ones) actually provide this information to the kernel. 
 
Even worse, some of them, e.g. an incredible Samsung SS805 SSD drive with firmware version `AD3Q` (MCCOE1HG5MXP-0VBD3-0H2) which I was lucky enough to own, are broken enough to deliberately report *wrong* information about `physical_block_size`, `logical_block_size` and most importantly `minimum_io_size` to the OS (this particular drive reported 8912 bytes as the `minimum_io_size`).

Therefore, it is not surprising at all, that right after partitioning the disk and formating the partitions, the installation program refused to mount the newly created file systems. At the first sight, of course, the OS, and more specifically `mkfs` were to get the blame. How come it creates file systems that are so broken that they can not be even mounted?

A more careful investigation revealed, however, that `mkfs` is just doing its job: the underlying `minimum_io_size` hint (8K) gets propagated upwards and so it creates a file system with a 8K block size without hesitation.

However, in order for `mount` to be able to mount the file system, the block size should be <= kernel page size (which is 4K on x86_64 under normal conditions) [lkml-1]. Hence, the file system can be created, but not used.

[lkml-1]: http://lkml.org/lkml/2006/9/8/4

Now that this has been figured out, all it takes is to find a firmware update to version `CD3Q` and magically all is well again... However, if you haven't been previously exposed to I/O limits-related issues, would you be able to make any sense out of the mysterious `EXT4-fs: bad block size 8192` messages in `dmesg`?

As an exercise for the readers, now forget everything that you have recollected so far and try to read the post backwards!

