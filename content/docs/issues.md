---
Title: Known Issues
---

Every computer system has problems, and Labinator is no exception - especially being powered by such cheap "junk" gear.
This page serves as a supplement for users of Labinator who would like to understand more about the dreaded "☠️It's taking too long! Something went wrong☠️" message.

More details of the scenario and causes will come.

## Pods that never start
Sometimes pods fail to start, causing Labwatch to throw a timeout
* Daemonsets (otelcol) cannot start on a node that doesn't exist
* Prometheus lives on whichever nodes have the pod PVC - so it cannot start on a node that doesn't exist

The root cause is one of the two issues documented following this one (VMs that were once up vanish, and sometimes our cheap junk nodes go into a tailspin)

## Disappearing VMs
This is a bit of an enigma.
Labwatch waits for all Talos nodes to be up before bootstrapping the cluster and then installing additional workloads.
From time to time, after a node comes up and joins the cluster, it vanishes.
As-in... libvirt on the physical node doesn't even show that it exists.

The root cause is *currently* unknown - it started cropping up way late in the build of Labwatch and I have not yet triaged.

This can be helped by [improving](/docs/improve/#vm-resilience) Labinator

## Freaked out nodes
Sometimes, the hypervisor nodes get into a hung state.
You know this is hitting a node when the graphical load display ([htop](https://htop.dev/)) shows all four CPU cores pegged at 100%.
At this point, the node is completely unresponsive over the network.

The root cause is *currently* unknown - it requires a physical keyboard attached to the machine for further triage.
It may not be worth triaging further since these cheap Celeron CPUs aren't really designed to run a type 2 hypervisor!

## Mysterious "network" issues
From time to time, all six nodes will report errors fetching boot assets from Boss.
What makes it especially mysterious is that the problem occurs across all six nodes and has been observed both during the TFTP phase of PXE (handled by [dnsmasq](https://en.wikipedia.org/wiki/Dnsmasq)) as well as the HTTP phase of iPXE.
Confusingly, shells and VNC sessions open to Boss at the time of failure do not have issues.
It is very likely that the problem exists somewhere on the Boss node rather than the network.
The network is as simple as it gets and Boss is just another cheap "junk" node that is serving DNS/DHCP/TFTP/HTTP for the boot process.

The root cause is *currently* unknown - triage attempts from Boss to the rest of the network (which isn't much) show no issues.
Boss can serve files to itself and the SSH/VNC (which ARE resilient to networking delays, so this may not mean much) remain available even when this occurs.