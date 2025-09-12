---
title: Statusinator
---

Code: [https://github.com/DRuggeri/labinator_statusinator/](https://github.com/DRuggeri/labinator_statusinator/)

Statusinator is the firmware managing an ESP32-powered board with a built-in IPS screen mounted in the bottom right of Labinator.
It's job in the lab is simple: be pretty by [showing the current status of... stuff](https://lopaka.app/gallery/7824/16192).
As can be seen in the above link (which is a totally awesome UI tool to mock up graphical displays), the power status of each port and log/dns/dhcp/firewall counts are displayed consistently.
The currently running lab gets a line showing the name, how many hypervisors are needed vs up, how many nodes are needed vs up, how many pods are needed vs up, and the current step of the lab.

From there, log lines are continuously scrolling as-received by [Labwatch](/docs/subprojects/labwatch/#event-marshalling-and-log-aggregation).

Additional documentation detailing the main control loop and pictures will come.