---
Title: About
---


## Goals and Principles
While the initial goal was pretty cut and dry (I forget the exact original use case - maybe having an easy place to create a K8S controller that synchronizes external data to configmaps or something), it quickly evolved to require a much broader need for experimentation. The very first version of Labinator was an old [Dell Optiplex 7050 micro](https://i.dell.com/sites/csdocuments/shared-content_data-sheets_documents/en/optiplex-7050-towers-technical-specifications.pdf?cjdata=MXxOfDB8WXww&cjevent=e7d7610aead411ef822300bc0a82b820&dgc=CJ&publisherid=5370367&publisher=&aff=Skimlinks&affid=5370367&aff_webid=100062990&aff_user_id=31285X1023697X1473eef2b4daec31877dad3a9139161d&gacd=9684992-28463632-5750457-345576786-177846717&dgc=af&VEN1=14349898-100062990-31285X1023697X1473eef2b4daec31877dad3a9139161d-Skimlinks&dclid=CIWKtZCew4sDFZv09QIdGlMlSw) (inspired by [Project TinyMiniMicro over at ServeTheHome](https://www.servethehome.com/introducing-project-tinyminimicro-home-lab-revolution/)) that I threw [Debian](https://www.debian.org/) on and stood up a K8s cluster [the hard way](https://github.com/kelseyhightower/kubernetes-the-hard-way). With all the messing around with daemons/etc, building atop a physical node was too cumbersome so [libvirt+KVM](https://libvirt.org/drvqemu.html) quickly made an appearence. It evolved rapidly from there as I played with more integrated approaches to K8S such as [Flatcar Container Linux](https://www.flatcar.org/), [Fedora CoreOS](https://fedoraproject.org/coreos/), and [Talos Linux](https://www.talos.dev/). The things we go through to ensure we choose the right stuff for our home labs...

The goals for Labinator became a build with these attributes:
* Cheap, commodity hardware that resembles datacenter-grade equipment
  * x64 compute
  * VLAN-capable network supporting IPv4 and IPv6
* ... in an isolated network
* ... with airgap operation
* ... using Open Source software
* ... that is flexibly set up for lots of experiments
* ... with good security on the data plane
* ... which can be rebuilt at any moment
* ... and is portable 😮

The cheap/commodity hardware is self-explanatory. Labs *should* be cheap and are often built from bare-minimum-spec gear. The need for an isolated network enables the ability to airgap the installation as well as to contain any... crazy that might happen in the lab. It also serves the portability need. As a advocate and lover of Open Source software, and not somone willing to shell out huge 💰 just to operate a lab, the usual FOSS stance became the default. Of course, rapid iteration is needed for any level of crazy experiment, so being able to nuke and pave the environment to get back to a known good starting point is a must.

Finally, it is worth pointing out that many of us technology leaders (nerds 🤓?) use our homelabs as precursor environments before introducing new technology at [work](https://www.linkedin.com/in/danielruggeri). It is therefore very important to me that the system as-configured meets security requirements one can reasonably expect in an enterprise. This is also why physical portability became important - I want to be able to *show* folks at work and [out on the conference circuit](https://events.linuxfoundation.org/kubecon-cloudnativecon-north-america/) this self-contained ecosystem.

Consider these goals as you wander deeper into the abyss of Labinator. Some will not apply to your use case and some will be seemingly silly for what you want to try/do. That's OK, rip that part out of your own lab/experiments.

### Why cheap hardware?
Simply put, cheap hardware can be replaced easily and it also stands the best chance of being replicated by anyone interested in building such a lab.
While the [models](../models) used to secure the lab to the board are designed specifically for THIS lab, the hardware used in the lab is easy to swap out with anything else that has equivalent functionality.

### Why an isolated network?
Labinator is designed to be used for several kinds of experiments.
Having a principle of being in an isolated network frees the lab itself from needing to worry about a supporting network.
Want to try out an IPv6 only LAN?
Looking to play around with [iPXE](https://ipxe.org/)?
Want to try something clever with your DHCP servers?
Interested in seeing what happens when network partitions occur?

These things and more are much more sane to set up when the lab is in its own isolated network.

### Why airgap operation?
Since the lab needs to be portable and because it is sitting in an isolated network, it makes sense to expect the lab to be functional without external network connectivity.
At the same time, I am experimenting with different technologies that I'd like to see used at work - and that environment is always airgapped.
By forcing the airgap requirement up front, it means the experiments need to be built in ways that are much more likely to succeed in a real world enterprise deployment.

### Why Open Source?
Yes, some of the experiments to run may end up in an enterprise environment, but aside from Labinator, I also run a fairly complex home lab.
Building Labinator in a way that doesn't rely on proprietary software or software that is not freely available means that the lessons learned (the configurations built) are able to be applied directly to my own home lab.
Additionally, many of the FOSS projects used in Labinator are best in breed such that they can reasonably be expected to exist in a well funded enterprise environment, too.

Beside all of this, I'm a big fan of Open Source anyway.
It's without a doubt that I've benefitted from FOSS participation more than I've ever been able to give!

### Why flexible configuration?
I don't know what I don't know.
The physical components of Labinator must be able to be composed in a number of ways that will support the *current* tests I want to run, but also to support testing use cases I haven't thought of.
This made it important to ensure that an actual router (rather than just switching) is in the lab.
Additionally, the network has to support VLANs to create arbitrary separations amongst the nodes.
Speaking of nodes, using a multiple of three ensures that nodes can be carved up in support of consensus-based protocols.

### Why security on the data plane?
Sure, to run experiments you may not need great security.
Sometimes it's enough to just see how things react when [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security) isn't used.
Sometimes it's fine if you want to just get a feel for software when the environment isn't airgapped.
That doesn't quite work for me, though...
We take security very seriously where I work.
Since a lot of the experiments I am running I hope to bring to $dayjob anyway, it makes sense to just bake security in from the beginning.
Besides, a lot of the lessons and best practices used at work get applied to my own network anyway.
Once you know how easy it can be to infiltrate/damage an environment and once you see the kind of havoc it can wreak, it's just impossible to ignore security in your own lab.

### Why an easy rebuild?
A principle I've encouraged at work for many of my teams has been the idea that anything we build should be able to be "nuked and paved".
This encourages automation of repeatable tasks which also creates consistency in outputs.
Even in my own homelab, I use extensive automation for managing the gear such that any machine can be replaced easily and "rehydrated" to be back in business without too much trouble.
At first, this may seem like a lot of work for little benefit... but I've been in the game long enough to know better.
Investing in automation and repeatability begins paying off after the first hardware failure - and you get returns every time you need to upgrade gear.

Setting philosophy aside, this is a lab - it's MADE to break!
Breaking things means fixing things, and after I've trashed a lab (accidentally or on purpose) I don't want to spend the time figuring out how to unbreak it.

### Why should it be portable?
I could say that I want Labinator to be portable for a handful of contrived reasons, but the main reason is because I want it to have the "wow factor".
Yes, I could record videos of the experiments being run or capture scripted output of what happens when a certain thing changes in the lab, but being able to have the lab do something in an unscripted fashion also just makes things more "real".
There's just something about seeing it all in action, in person, that demystifies the technology at play.

With [labwatch](../subprojects/labwatch) monitoring and controlling Labinator, it's also possible to trigger various scenarios based on your own heart's whims.
Want to create a partition between nodes in a Kubernetes cluster? Give it a shot!
Want to do the same thing, but with the lab set up as a few baremetal nodes running something else? Sure!
Having that flexibility coupled with the isolated network really begs for the lab to be physically right in front of you.

Which is cool and all... but it's still really *awesome* to build such a thing that can be dragged around to show people!

## What it isn't
Although Labinator may serve as a reference implementation suitable for your environment's foundation, it does not try to be all things to all people. Some compromises were made for the sake of operability and experimentation. A few key things to call out:

* **Perfectly secure data plane** - A principle of Labinator is for the data plane of the experiment to be reasonably secure. But "reasonably secure" is going to mean different things to different people. Your security posture may require encrypted private keys, exotic ciphers, private keys stored on a [HSM](https://en.wikipedia.org/wiki/Hardware_security_module), or other variations that are not addressed in Labinator.
* **A well-secured environment** - Because Labinator is in an isolated network and can operate airgapped, compromises have been made for the ecosystem *surrounding* the experiments. While some security aspects needed for the data plane to be reasonably secure bled into the surrounding environment (e.g. TLS for container images/log streams), other aspects have been knowingly left more simple. Some examples:
  * Administrative credentials for the router/switch/server/etc are weak. This is to make it easy to get in there and poke around
  * While the data plane is configured with [PKI](https://en.wikipedia.org/wiki/Public_key_infrastructure), it is set up using a local (to the lab) internal [CA](https://en.wikipedia.org/wiki/Certificate_authority). Things inside the lab need to trust that CA - but that CA shouldn't be particularly "trusted" outside of the lab context given that it uses a self-signed root with a key stored in software that can easily be exfiltrated.
  * Daemon best practices on the boss node are mostly ignored. Each daemon ideally should run as its own user in its own container isolated from other daemons and next to nothing should run as root user. That's not particularly hard to do, but it makes the setup infra code more difficult to grok - besides, the environment is designed to be destroyed/recreated on a whim.
* **Always the best place to experiment** - The Labinator works well for the things I want to test out, but it may not be perfect for what you are trying to test or experiment with. As a compute-centric lab, it's good for experiments with bare metal equipment, virtualization, containers, etc. However, it lacks many of the important features that would be needed for novel network configurations beyond simple VLAN separation.