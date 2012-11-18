---
layout: site
title: UCF VPN in Linux (KDE)
subtitle:
---
I figured I'd make a quick blog post about how to connect to the University of Central Florida's VPN in Linux (kubuntu 11.10, to be precise) for anyone who has a need to, since I had to figure this out. UCF recently changed their VPN software over to Cisco AnyConnect VPN, which works in Windows/Mac, but not Linux -- so these instructions should work for at least the next few years.
 
First, make sure you have the Cisco VPN provider for Linux, ``vpnc``, installed.

    sudo apt-get install vpnc network-manager-vpnc

Next, make sure you know your NID username. You can look it up with [this form](https://my.ucf.edu/nid.html).
 
Then, make sure you know your NID password. You can reset it with [this form](https://www.secure.net.ucf.edu/extranet/reset/validation.aspx?type=nid). (You need to know your PID for this one, which you can look up with [this form](https://my.ucf.edu/pid.html).)
 
Now, click on your networking system tray icon and click the "Manage Connections..." button. Then go to the VPN tab and click the "Add..." button and choose "VPNC".

Fill out all of the fields as they are in the screenshot below. Note that if you are a student and not faculty or staff, you need to set the group to "ucfstudent" and the group password to "knights", instead.

![Screenshot](/images/2011-12-16-ucf-vpn.png)

And that should give you access to everything as if you were in campus.