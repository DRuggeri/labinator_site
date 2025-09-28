---
Title: What's New?
Weight: 2
---

I didn't quite plan to turn this little mini-site into a running blog or capture a CHANGELOG, but as it turns out... I keep playing with this thing and improving it just for fun.
I haven't given any thought to the idea of versioning or generations of Labinator, so this list more or less is an ad hoc pseudo-changelog that captures realtively noteworthy additions as I work on it.

### Mid '25
After launching Labinator, some stuff didn't work well or at all (notably: the first on in the list below was a breaker) so I lugged the lab back to my lab so I could hack on it some more.
* Fix - Expired certs [due to not being used](../improve/#expired-certs). This thing was designed to be played with *all the time*
* Create a way to [add/change application load](../improve/#add-load) in Kubernetes
* Show more [node-specific stuff](../improve/#status-page-node-alignment) on the status page
* Fix - Dramatically [increase reliability](../improve/#boot-up-reliability) of launching labs
* Make it easier to [lug Labinator around](../models/carry-handle/)
* Squash a bunch of edge case concurrency bugs in labwatch
* Add the ability to inject network failures into the lab for optimal chaos
* Create a [Kubernetes watcher](https://github.com/DRuggeri/labinator_labwatch/commit/1894b9112b614404f0732d5cb675c9644abdfb45) so we can see pod information (like where they run)
* Stitch labwatch [directly into the OTel collector log stream](improve/#log-stream-reliability) rather than Loki
* Redesign the power wire [top](../models/power-wire-cover-top/) and [bottom](models/power-wire-cover-bottom/) mounts because the stupid [front kept falling off](https://www.youtube.com/watch?v=3m5qxZm_JqM)
* Add a watcher for LAN/WAN in and out packets and push the stats to [statusinator](../subprojects/statusinator/)