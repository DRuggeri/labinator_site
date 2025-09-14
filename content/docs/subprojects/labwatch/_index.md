---
title: Labwatch
---

Code: [https://github.com/DRuggeri/labinator_labwatch/](https://github.com/DRuggeri/labinator_labwatch/)

Labwatch is the the main controller dude of Labinator. It deals with orchestration of the labs, tracking the progress of labs, gathering and exporting events/status, and presenting the user with the screens on the lefthand side touch screens. Without labwatch, there isn't much one could do automatically in the lab. Architecturally, it goes overboard with go channels to decouple components and try to maintain a semblence of separation of concerns.

### Web Interface

The main entry point is the touchscreen interface served from [site/index.html](https://github.com/DRuggeri/labinator_labwatch/blob/main/site/index.html), which lets you select labs, launch experiments, trigger chaos events, and observe the mayhem. The [progress.html](https://github.com/DRuggeri/labinator_labwatch/blob/main/site/progress.html) page provides a handy slideshow that what should be happening during lab initialization, complete with auto-advancing slides that explain each step and a watcher that makes sure when major steps change, the slideshow updates. While labs are launched, the [loadstatus.html](https://github.com/DRuggeri/labinator_labwatch/blob/main/site/loadstatus.html) page gives a visual layout of all the physical nodes, their power states, virtual nodes, and container counts in real-time. After startup, it automatically switches to display the load throughput of the clients and servers in the cluster... which is fun for figuring out what affect chaos actions create on running workloads!

Give the main page a try here. Obviously nothing happens when you click the links, but you get the idea!
{{< iframe src="https://labinator.bitnebula.com/labwatch/" width="512px" height="300px" title="Labwatch Main" >}}

### Event Marshalling and Log Aggregation

The log watching system is beautifully over-engineered. The [`loki.LokiWatcher`](https://github.com/DRuggeri/labinator_labwatch/blob/main/watchers/loki/lokiwatcher.go) connects via WebSocket to Loki and normalizes incoming log events, while the [`otelfile.OtelFileWatcher`](https://github.com/DRuggeri/labinator_labwatch/blob/main/watchers/otelfile/otelfilewatcher.go) tails the OpenTelemetry collector's JSON log files. In practice, the Loki watcher is disabled because Loki is receiving all the same events as OTelCol - and reading a file has way lower overhead. Both feed into a common [`common.LogEvent`](https://github.com/DRuggeri/labinator_labwatch/blob/main/watchers/common/types.go) stream that gets processed, batched for performance, and distributed to all the interested parties - including the [`statusinator.Statusinator`](https://github.com/DRuggeri/labinator_labwatch/blob/main/statusinator/statusinator.go) for that sweet scrolling log display on the bottom right-hand screen.


### Relayinator Power Management

The [`powerman.PowerManager`](https://github.com/DRuggeri/labinator_labwatch/blob/main/powerman/manager.go) talks to [relayinator](/docs/subprojects/relayinator) over serial to toggle power buttons on all the nodes. There's also a background job that polls relayinator to know the current state of each power port. With this little trick, labwatch can know if you've manually toggled power on relayinator (but if you press the real button on the computer, it's anyone's guess).

### The Initialization Dance

The [`talosinitializer.TalosInitializer`](https://github.com/DRuggeri/labinator_labwatch/blob/main/talosinitializer/cmd/main.go) orchestrates the entire lab startup process - wiping disks, configuring network boot links, powering on nodes in the right sequence, and waiting for Talos to bootstrap the Kubernetes cluster. It's like conducting an orchestra where half the instruments are cheap eBay gear that might decide to take a nap at any moment.

### Window Management Wizardry

The [`wm.WindowsManager`](https://github.com/DRuggeri/labinator_labwatch/blob/main/wm/manager.go) controls what appears on the bottom screen, launching everything from `htop` and `tcpdump` to VNC viewers for individual VMs. Because sometimes you just need to see what's actually happening inside the mess of a lab. These are triggered by Observe requests from the top screen.

### The Army of Watchers

Each watcher is a self-contained struct with goroutines that specialize in monitoring a specific aspect of the lab infrastructure. They all feed status updates into Go channels, which eventually make their way to the central status aggregator. They're purpose-built sensors that speak a common language - they observe, normalize their findings into structured data, and broadcast it back to the main labwatch loop (or into the void, if things get too overloaded).

The beauty of this design is that each watcher can fail, restart, or go completely bonkers without taking down the whole system. They're like independent reporters covering different parts of the unfolding lab.

#### Status Aggregation

The [`statushandler.StatusWatcher`](https://github.com/DRuggeri/labinator_labwatch/blob/main/handlers/statushandler/handler.go) acts as the central nervous system, collecting status updates from all the watchers and broadcasting a unified [`common.LabStatus`](https://github.com/DRuggeri/labinator_labwatch/blob/main/watchers/common/types.go) to anyone who wants to listen. This includes power states, log statistics, Talos node health, Kubernetes pod counts, port availability, and initialization progress - basically everything you need to know if your lab is behaving or having an existential crisis. These status updates and aggregated details are displayed on both [Statusinator](/docs/subprojects/statusinator) and the loadstatus page.

#### Port Watcher

The [`port.PortWatcher`](hhttps://github.com/DRuggeri/labinator_labwatch/blob/main/watchers/port/portwatcher.go) is the simplest guy of the group. While other watchers deal with APIs and fancy protocols, the port watcher just tries to connect to TCP ports and reports back whether stuff is actually working.

Its genius lies in its simplicity and persistence. It continuously probes a configurable list of endpoints (APIs or whatever answers on TCP) and provides the simple indicator of whether services are up or down on that port. For an API-centric lab with things like Talos and Kubernetes that don't answer until they're ready, this turns out to be a reliable indicator of when work can commence.

#### Talos Watcher

The [`talos.TalosWatcher`](https://github.com/DRuggeri/labinator_labwatch/blob/main/watchers/talos/taloswatcher.go) is probably the most sophisticated member of the watcher family. It connects to each Talos node's API and streams real-time events about boot sequences, service states, and cluster membership changes. 

What makes it clever is how it handles the inherently unreliable nature of nodes that might be powered off, rebooting, or having network issues. It maintains persistent connections when possible but gracefully degrades to polling when things get sketchy. The watcher also understands Talos's complex boot phases and can tell you exactly where each node is in its journey from "cold metal" to "productive cluster member."

#### Kubernetes Watcher

The [`kubernetes.KubeWatcher`](https://github.com/DRuggeri/labinator_labwatch/blob/main/watchers/kubernetes/kubewatcher.go) keeps an eye on the Kubernetes cluster's pod ecosystem. It's not just counting pods - it's tracking their lifecycle states across different namespaces - and broken down by which node they're running on.

The clever bit is how it aggregates pod information by node and namespace, giving you both macro and micro views of cluster health. It can tell you if something wonky has left all your pods scheduled on the same node, if a pod is stuck in a crash loop or the pod otherwise cannot start, or if the cluster is healthy enough to handle whatever chaos experiment you're about to unleash upon it.

#### Callbacks Watcher

The [`callbacks.CallbackWatcher`](https://github.com/DRuggeri/labinator_labwatch/blob/main/watchers/callbacks/callbackwatcher.go) is the diplomatic channel for workloads that want to report their own status. It runs an HTTP server that receives webhooks and status updates from running applications.

This is slick for allowing a thing to arbitrarily pop up and provide a key/value pair. Instead of trying to guess where we are in the lifecycle status of what a hypervisor is doing from the outside, the callback watcher lets the boot process speak for itself. Load testing clients can report their progress, reliability tests can announce their completion, and custom workloads can provide detailed status updates that no external monitoring could infer.

### Handler Ecosystem

The HTTP handlers are simultaneously the face and backoffice of labwatch - they provide REST API and WebSocket endpoints that everything else uses to interact with the lab. Each handler is a specialized HTTP endpoint that knows how to translate web requests into the appropriate internal operations.

What makes them particularly clever is how they maintain state and coordinate with the various subsystems without getting tangled up in each other's business. They're designed to be stateless where possible, but when they need to maintain context (like ongoing reliability tests), they do so in a clean, observable way... often with many mutexes.

#### Browser Handler

The [`browserhandler.BrowserHandler`](https://github.com/DRuggeri/labinator_labwatch/blob/main/handlers/browserhandler/handler.go) is the puppet master for the Firefox instance running on the bottom screen. It is used to navigate to different URLs, refresh pages, and essentially turn the display into a dynamic dashboard that changes based on what's happening in the lab.

#### Event Handler

The [`eventhandler.EventHandler`](https://github.com/DRuggeri/labinator_labwatch/blob/main/handlers/eventhandler/handler.go) manages the WebSocket connections that provide real-time event streaming to browsers and other clients. This is especially useful for dynamically attaching and detaching to the log stream with an HTTP client to see what is going on without needing to load up Loki or to hop on (Boss)[/docs/subprojects/boss] to look at logs.

#### Scaler Handler

The [`scalerhandler.ScalerHandler`](https://github.com/DRuggeri/labinator_labwatch/blob/main/handlers/scalerhandler/handler.go) is your gateway to dynamic workload management. It lets you add or remove client and server workloads on the fly, which is perfect for demonstrating load behaviors based on chaos or testing how the cluster responds to varying load patterns.

#### Load Stat Handler

The [`loadstathandler.LoadStatHandler`](https://github.com/DRuggeri/labinator_labwatch/blob/main/handlers/loadstathandler/handler.go) is the data aggregation powerhouse for load testing scenarios. It collects metrics from load [clients](https://github.com/DRuggeri/labinator_chef/tree/main/files/counterclient) and [servers](https://github.com/DRuggeri/labinator_chef/tree/main/files/counterserver) and provides consolidated reports that show how well the cluster is distributing synthetic traffic. This is all visualized on the loadstatus page as well!

#### Reliability Test Handler

The [`reliabilitytesthandler.ReliabilityTestHandler`](https://github.com/DRuggeri/labinator_labwatch/blob/main/handlers/reliabilitytesthandler/handler.go) repeats the virtual-2 lab over and over and over again. As it does so, it keeps track of which steps have failed, which steps have timed out, and aggregate stats about how many labs have run/failed over time. This handler was used as "burn in" for labinator to suss out what parts of the system are the most unreliable as well as to let issues surface over time with lots of labs being run on a cycle.

All of this runs as a single binary configured via [YAML](https://github.com/DRuggeri/labinator_chef/tree/main/recipes/labwatch.rb), because if you're going to over-engineer something, you might as well make it configurable too. The whole thing is held together with Go channels, WebSockets, and what can only be described as an unreasonable amount of optimism about cheap hardware reliability.