---
layout: post
title: PC turns off on high CPU usage
---

A few days ago I accessed my PC BIOS to enable virtualization support(AMD SVM), after a reboot I kept using the system with no
issues at all until I executed a software called BOINC that runs CPU intensive tasks, I realized that after running such software
the system shutdown, at a first moment I thought it was some strange bug on BOINC forcing shutdown, so after rebooting I decided
to do not run BOINC but to play Apex Legends, one minute of gameplay led to system shutdown, at this point I
realized it was a hardware issue. Rebooted again and used a software called Speccy to monitor components temperature and executed
BOINC, in a matter of seconds CPU temperature went from 46ºC to over 70º. With al this happening I accessed BIOS in an attempt to
find what was going on and realized that had changed an option called EZ System Tuning from Normal to ASUS Optimal, after reverting
this configuration my computer worked fine. I believe this option should only be enabled by those who have a more powerful cooling
system and not a cooler box like me. The moral of the story is to always be careful with BIOS configurations.

By the way if you are facing some similar issue my components are:

 - Motherboard: ASUS B350M-A
 - CPU: AMD Ryzen 5 1400(stock)
 
And here are two photos showing this option.

![EZ System Tuning Normal](/images/ez_system_tuning_normal.jpg "EZ System Tuning Normal")
![EZ System Tuning ASUS Optimal](/images/ez_system_tuning_asus_optimal.jpg "EZ System Tuning ASUS Optimal")
