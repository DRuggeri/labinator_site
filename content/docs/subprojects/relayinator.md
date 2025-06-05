---
title: Relayinator
---

Code: [https://github.com/DRuggeri/labinator_relayinator/](https://github.com/DRuggeri/labinator_relayinator/)

Relayinator is the firmware managing a physical piece of gear (an ESP32-powered board with eight relays) and the hardware build to accomplish managing power status within the lab.

Physically, Relayinator wraps a [bottom](/docs/models/relayinator-bottom/) and [top](/docs/models/relayinator-top/) 3D print around a [commodity board](/docs/bom/).
The bottom features holes for standard 5mm x 2.1mm barrel connectors across two rows:
* Bottom: Direct feed from PSU - always on
* Top: Relay-controlled ports
I went through a few designs, but ultimately landed on the bottom ports all being 12v except the far left (power for the router) and far right (power for the monitors) being 5v.
The top ports, which are relay-controlled (AKA: switched on/of when I want them to be) connect to the seven nodes of the lab.
Ports 1 - 6 correspond to Nodes 1 - 6, port 7 is for Boss, and port 8 is free (eventually, it was going to control power to the monitors).

More documentation including wiring diagrams will come