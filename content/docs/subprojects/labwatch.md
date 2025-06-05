---
title: Labwatch
---

Code: [https://github.com/DRuggeri/labinator_labwatch/](https://github.com/DRuggeri/labinator_labwatch/)

Labwatch is the main lab controller dude. It has the following responsibilities:
* Serve the sites for the top touchscreen in Labinator (index and progress)
* Initiate the execution of a lab based on selections
* Watch things happening across Labinator
  * Tailing and distributing Loki logs (websocket)
  * Aggregating status and distributing it to clients (websocket)
  * Communicating with [Relayinator](/docs/subprojects/relayinator) to observe/manage power state of the nodes
  * Communicating with [Statusinator](/docs/subprojects/statusinator) to push status and logs to the bottom screen
  * Monitoring up/down status of TCP ports during the lifecycle of a lab
  * Receiving HTTP callbacks and updating internal statuses from labs
* Communicate with Firefox on the bottom screen to change URLs as needed in the lab
* Communicate with the system to launch programs on the bottom screen as needed
* Orchestrate the initialization process for Talos

Architecturally, it goes overboard with go channels to decouple the components.

Future documentation will be added.