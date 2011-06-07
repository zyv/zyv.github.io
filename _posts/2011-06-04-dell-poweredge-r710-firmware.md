---
layout: post
title: Updating firmware on Dell PowerEdge R710
---

Introduction
------------

Updating firmware used to be an uncomplicated, albeit slightly dangerous process. Unfortunately, as the server hardware developed over time, it did not become any simpler, but rather evolved into even more of an arcane rite.

Moreover, it seems that there is no comprehensive checklist regarding the firmware updates of the Dell PowerEdge server family and the information is spread over the support forums, tech wikis and miscellaneous blogs. Hence, this compilation was created in a hope that it will be useful and save time to some.

Let us consider a case of a Dell PowerEdge R710 featuring 2 x 2T hard drives and a 100G solid state drive, equipped with an iDRAC 6 Enterprise remote access card. There are generally several avenues that one might take to update the server firmware:

*   iDRAC firmware update facility (limited to the firmware of the DRAC itself and the USC, Dell Universal Server Configurator / Lifecycle Controller [dell-1]), which requires manually downloaded firmware update (or repair) packages
 
*   Dell USC / LC, which is an UEFI based software that is able to update almost all firmwares of the devices that are present in the server using a number of possible sources of updates:
 
    -   Dell FTP site [dell-2], in which case at least one of the network interface cards present in the system has to be configured from within USC, so that the server would be able to access external resources directly or via a proxy server
    
    -   Special USB media prepared beforehand using a software called Dell Repository Manager [dell-3], which needs to be deployed to a Windows management PC
    
    -   Dell OpenManage Server Update Utility [dell-4], which is a DVD image containing a comprehensive collection of firmware updates; in some sense, the USB media produced by the Repository Manager are subsets of SUU specific to each particular server
 
*   Dell Systems Management Tools and Documentation all-in-one DVD [dell-5], which contains the whole suite of OpenManage-branded Dell systems management software (OMSA, SBUU, SSDT and ITA)
 
*   Dell Update Packages (DUPs), which are accessible from the download pages for each specific Dell server [dell-6]; the right download page can be found by entering the service tag (unique server family identifier, which can be found in the DRAC interface or USC among other sources)

[dell-1]: http://support.dell.com/support/edocs/software/smusc/
[dell-2]: ftp://ftp.dell.com
[dell-3]: http://support.dell.com/support/edocs/SOFTWARE/smdrm/
[dell-4]: http://support.dell.com/support/edocs/software/smsuu/
[dell-5]: http://support.dell.com/support/edocs/software/smsom/
[dell-6]: http://support.dell.com/

A rather complete (but possibly not exhaustive) list of components that might require updates is as follows:

*   iDRAC card
*   Dell BIOS
*   Dell USC / LC
*   Dell OS Drivers pack (part of LC)
*   Dell 32-bit Diagnostics software (part of LC)
*   Dell-branded RAID controllers, i.e. PERC H200I
*   Network interface cards
*   SSD devices

What follows are comments regarding each one of those and applicable update methods.

Possible firmware update procedures
-----------------------------------

### iDRAC firmware update

Updating the DRAC firmware should be the first step to updating anything else, especially in the case of a restricted onsite presence / remote hands availability. During the update, the DRAC might become unavailable for a period of time up to 15 minutes; this is normal and expected.

It is preferable to turn off the server and perform the update from the DRAC management console (iDRAC Settings | Update | Upload). The update package has to be downloaded manually from the server support and drivers home page at Dell.

Additionally, iDRAC is able to re-flash USC / LC using USC repair packages [lc-1] in the case if it was hosed during the update. It is generally not recommended by Dell and considered to be a last-resort action, but I have found it to be the only reliable way of updating the USC.

[lc-1]: ftp://ftp.dell.com/LifecycleController/

### Dell USC / LC Platform Update

This was found to be the most reliable and complete procedure to update most of the firmwares in the system. In order to perform such an update, one needs to enter the "System Services" menu on boot (F10).

Once USC is loaded, the Platform Update can be launched. It is recommended to first update all of the suggested firmwares and then again launch Platform Update to update USC itself separately, because it was found to be the less reliable update of all.

Sometimes, the USC will get stuck after update and the server will keep displaying "Entering USC..." message for hours (everything below 20 minutes might still be fine). In this case, USC can be restored via iDRAC as described above.

Unfortunately, it took a lot of experimentation to find out that this particular server will not handle anything older or newer, but USC 1.3. There is no scientific explanation for that up to now, because generally in such a case Dell just replaces the motherboard without going on into the details of what exactly went wrong.

### Dell Systems Management Tools and Documentation

Using this DVD instead of downloading the tools separately is important, because the SMTD DVD is bootable, unlike those on which the rest of the tools are distributed, where one is expected to prepare bootable disks oneself.

The process is rather trivial: one needs to boot off the DVD and launch the platform update process. The update program will request the DVD to be replaced with SUU DVD or another media containing the update repository.

It is worth to note, that SUU generally contains most outdated firmware out there, so many updates might just not be available if one goes down this route.

### Dell Update Packages

DUPs are listed last, because they are expected to be run from a production operating system. Some DUPs, however, contain an ISO generator to create a bootable image which does not require an operating system to be installed, but it is not always the case.

Dell normally provides DUPs on the server support pages for Windows and Linux or upon request. There is a community-supported repository [dell-linux] with DUPs wrapped around with native Linux packages, however, it is not officially endorsed or supported.

[dell-linux]: http://linux.dell.com/wiki/index.php/Repository

Conclusion
----------

It is worth to note, that some updates (even critical ones!) are only available in form of DUPs, i.e. for the SSD devices and not even referenced from server support pages. In such cases, one needs to search Dell support website [dell-support] using the model or serial number as the keyword.

[dell-support]: http://support.dell.com/

These updates are not to be neglected as for instance a recent critical SSD update was to fix the minimum advertised I/O block size.

