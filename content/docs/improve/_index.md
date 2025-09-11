---
Title: Things To Improve
---

## PXE and iPXE
For this combination of gear, PXE and iPXE booting leaves a lot to be desired.
At the start of each lab, it takes something like 30 seconds to initialize PXE and to chain boot into iPXE.
iPXE then takes about 30 seconds to configure.
A 1 minute delay to boot nodes that start in less than a minute sucks.

So why even use network booting?
It boils down to solving the "moment 0" node bootstrapping problems.

### Hardware
Let's ignore virtual machines for a moment.
We have the issue of needing to boot our cheap gear with a Talos disk image *OR* a hypervisor disk image depending on the lab being launched.
There are limitations at hand as we noodle on how to approach this.
The biggest constraint is identifying which Linux distribution to boot (Debian for hypervisor or Talos for baremetal) early enough in the boot process.
For enterprise systems, this would be a pretty trivial issue.
We would use [IPMI](https://en.wikipedia.org/wiki/Intelligent_Platform_Management_Interface) capabilities available in an out-of-band management network to boot the machine from whichever disk image we want.
I confirmed that Talos boots fine from a USB drive (either physical or virtually connected) and the Debian hypervisor is already a live image, so that part is easy enough.

... but these aren't enterprise systems.
They are cheap, commodity, near-garbage systems because we do things the hard way in Labinator (and frankly, not every home labber or company wants to invest in the cost and infrastructure of IPMI-enabled gear).
So with IPMI out, what is left?

We *could* create a custom bootloader on a USB drive attached to the boxes.
In theory it would not be especially hard - build a very very very small Linux disk image with both OS images baked into it.
[GRUB](https://www.gnu.org/software/grub/) makes this trivial to do such that we could install a GRUB config on a USB drive that supports booting from one of the two (or more) options.
Seems like a tidy solution, but at what cost?
We need to immediately worry about the resilience of the USB drives attached to the USB ports.
USB keys are notorious for being shoddy and unreliable.
While there are tougher models out there, Labinator is built on the [principle](/docs/about/#goals-and-principles) of cheap, commodity hardware - so I wouldn't invest a ton of time into researching USB keys for resilience (I have already, actually, but just not for Labinator - cheap SDXC micro SD cards with a USB adapter can be pretty good).
But we also have the question of how much of a PITA this would be, and how we maintain the content (you ARE updating your base images regularly... right?!) of the physically attached media.
This adds some requirements to our mini bootloader.
The bootloader would ALSO, somehow, need to determine which image it should boot from some command and control server (like labwatch).
There are [creative solutions out there](https://chris.boyle.name/blog/2023/04/controlling-grub-from-the-network/), but we also want to stick to enterprise-like capabilities.
So we need a reliable, modifiable, network-manageable bootloader that doesn't introduce much of a maintenance burden.
* We could solve the "reliability" concern with better gear
* We could solve the "modifiable" concern by building a small live OS image that manages the contents of the USB key
* We could solve the network-manageable concern with some clever tricks in GRUB

... or we could just use network booting, which has been around for ages and is a tried-and-true method for solving exactly these problems!
Let's not reinvent the wheel - and other portions of Labinator require obvious things like an HTTP server from which images can be pulled.
In fact, there are some neat tools out there like [matchbox](https://matchbox.psdn.io/) that make managing this via configuration management systems much easier.
Labinator ended up taking this approach.
Initial [experiments with matchbox](https://github.com/DRuggeri/labinator_chef/blob/main/recipes/matchbox.rb) showed promise, but the flexibility provided by matchbox wasn't needed for our purposes (minor dig: the matchbox documentation is pretty weak for figuring out why something doesn't work).
Instead, chainloading PXE to [iPXE](https://ipxe.org/) and [tossing iPXE scripts in a known HTTP server location](https://ipxe.org/) was all that was needed.


### Software
This one is less of an issue, but very much worth documenting/noting.
When a Talos node boots up, it must be fed configuration information.
While the tiny disk image contains everything needed to become a control plane or worker node, it still needs to know information about the cluster that is forming.
There are two ways to provide this information.
* From outside in by [applying configuration](https://www.talos.dev/v1.10/introduction/getting-started/#apply-configuration)
* From inside out by [using a kernel argument](https://www.talos.dev/v1.10/reference/kernel/#talosconfig)

Both of these require the node to be alive on the network first - either to push configs in or to fetch configs.
In an enterprise environment (and a well-managed homelab), it's reasonable to expect that the node will have a DHCP reservation or some kind of service to discover the IP of a node that is being bootstrapped, so network connectivity shouldn't be an issue.
Besides, we're already netbooting.

The question then gets to be: which approach is better and why?
From a security perspective, pushing data into the node from a trusted external location to the nascent node is far better.
Think about the contents of the machine config file.
It contains cluster join tokens, CA keys, leaf keys, etc - a rich target.
Even if they are only available on the network for a short period of time, one must consider how this sensitive data is protected while still made available to the node.
IP ACLs are the obvious solution, but now you have to tickle/orchestrate an HTTP server during bootstrapping to limit the potential for exposure.
More moving pieces == more chances for things to go wrong.

In the case of Labinator, we took the easy way out by hosting the talos config next to the iPXE boot script.
There isn't much in the way of grand reasoning behind this - I had already built labs and tested by pushing configs in (hence the code already being present, but untested in labwatch) from the outside so with Labinator I wanted to try the pull method.

Improvement: Enable and test [this code](https://github.com/DRuggeri/labinator_labwatch/blob/b10e6c0731aeade766b6e700c494a00785316aa2/talosinitializer/talosinitializer.go#L355) to push configs in rather than fetch them.

Additional notes:
* You can ALSO pass talos configuration in via the hypervisor. In testing on vSphere, this works fantastically... but it was a disaster when I tried to do this with KVM and CoreOS (why isn't [this](https://github.com/coreos/ignition/blob/2ef5a3a86fa099a19ccae91dba08a60492c90673/internal/providers/qemu/qemu_fwcfg.go#L100) documented somewhere?!?!), so I assumed Talos would suffer the same quadratic time fate reading "firmware". It would be good to test this for VMs since I flipped from `--virt-type qemu` to `--virt-type kvm` in Labinator.


## Power Buttons

## Code Quality

## VM Resilience

## Labinator Is Too Big

## Weak Printed Mounts

## Status Page Node Alignment

## Policy-based Airgap

## Wireguard VPN
It's really cool to have labinator out and about doing stuff, but shortly after it went live, an unanticipated failure occurred: It was left off for a couple of days (unbelievable, right?!?). The surprise came when labinator was powered back up again: The [statusinator](/docs/subprojects/statusinator/) wasn't updating logs and counts. The root cause was simple to figure out (labinator uses [certs that are only valid for 24 hours](https://github.com/DRuggeri/labinator_chef/blob/main/recipes/step-ca.rb), so we just needed to [make sure the certs were still good any time a service using them was started](https://github.com/DRuggeri/labinator_chef/commit/81c179a0dcbe25efa9d220ae5541f9b6e06bdceb)).

What wasn't simple was debugging the problem. I had to lug the board full of junk back home, plug it into the network and hop into take a look. That's no good. Instead, an enhancement to Labinator for the future will be to configure [the firewall](/docs/layout/#wally) to establish a VPN tunnel, when network is available, to a server that I can use to remotely access labinator.

I started to play with this down in the lab, ultimately getting wireguard set up on the firewall, but failed to properly route traffic through it. This resulted in an even worse hack: dropping the uplink down into the [switch](https://labinator.bitnebula.com/docs/layout/#switch) so I could access the firewall from the inside - oops! No worries. I got it fixed again before I left town, but have yet to get back to it.


## Add Load - Done
A lab that allows you to stand up Kubernetes clusters is cool - but you kinda want to see Kubernetes do something other than monitor itself. To fix this, some simple little clients and servers were created as well as a [load manager for labwatch](https://github.com/DRuggeri/labinator_labwatch/commit/0530956c90afbdd6bcf21cb630b071dbe8e415ad) that allows the user to scale up or down the clients and servers dynamically.