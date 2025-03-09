---
Title: About
---


## Goals and Principles
While the initial goal was pretty cut and dry (I forget the exact original use case - maybe having an easy place to create a K8S controller that synchronizes external data to configmaps or something), it quickly evolved to require a much broader need for experimentation. The very first version of labinator was an old [Dell Optiplex 7050 micro](https://i.dell.com/sites/csdocuments/shared-content_data-sheets_documents/en/optiplex-7050-towers-technical-specifications.pdf?cjdata=MXxOfDB8WXww&cjevent=e7d7610aead411ef822300bc0a82b820&dgc=CJ&publisherid=5370367&publisher=&aff=Skimlinks&affid=5370367&aff_webid=100062990&aff_user_id=31285X1023697X1473eef2b4daec31877dad3a9139161d&gacd=9684992-28463632-5750457-345576786-177846717&dgc=af&VEN1=14349898-100062990-31285X1023697X1473eef2b4daec31877dad3a9139161d-Skimlinks&dclid=CIWKtZCew4sDFZv09QIdGlMlSw) (inspired by [Project TinyMiniMicro over at ServeTheHome](https://www.servethehome.com/introducing-project-tinyminimicro-home-lab-revolution/)) that I threw [Debian](https://www.debian.org/) on and stood up a K8s cluster [the hard way](https://github.com/kelseyhightower/kubernetes-the-hard-way). With all the messing around with daemons/etc, building atop a physical node was too cumbersome so [libvirt+KVM](https://libvirt.org/drvqemu.html) quickly made an appearence. It evolved rapidly from there as I played with more integrated approaches to K8S such as [Flatcar Container Linux](https://www.flatcar.org/), [Fedora CoreOS](https://fedoraproject.org/coreos/), and [Talos Linux](https://www.talos.dev/). The things we go through to ensure we choose the right stuff for our home labs...

The goals for labinator became a build with these attributes:
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

Consider these goals as you wander deeper into the abyss of labinator. Some will not apply to your use case and some will be seemingly silly for what you want to try/do. That's OK, rip that part out of your own lab/experiments.

## What it isn't
Although labinator may serve as a reference implementation suitable for your environment's foundation, it does not try to be all things to all people. Some compromises were made for the sake of operability and experimentation. A few key things to call out:

* **Perfectly secure data plane** - A principal of labinator is for the data plane of the experiment to be reasonably secure. But "reasonably secure" is going to mean different things to different people. Your security posture may require encrypted private keys, exotic ciphers, private keys stored on a [HSM](https://en.wikipedia.org/wiki/Hardware_security_module), or other variations that are not addressed in labinator.
* **A well-secured environment** - Because labinator is in an isolated network and can operate airgapped, compromises have been made for the ecosystem *surrounding* the experiments. While some security aspects needed for the data plane to be reasonably secure bled into the surrounding environment (e.g. TLS for container images/log streams), other aspects have been knowingly left more simple. Some examples:
  * Administrative credentials for the router/switch/server/etc are weak. This is to make it easy to get in there and poke around
  * While the data plane is configured with [PKI](https://en.wikipedia.org/wiki/Public_key_infrastructure), it is set up using a local (to the lab) internal [CA](https://en.wikipedia.org/wiki/Certificate_authority). Things inside the lab need to trust that CA - but that CA shouldn't be particularly "trusted" outside of the lab context given that it uses a self-signed root with a key stored in software that can easily be exfiltrated.
  * Daemon best practices on the boss node are mostly ignored. Each daemon ideally should run as its own user in its own container isolated from other daemons and next to nothing should run as root user. That's not particularly hard to do, but it makes the setup infra code more difficult to grok - besides, the environment is designed to be destroyed/recreated on a whim.
* **Always the best place to experiment** - The labinator works well for the things I want to test out, but it may not be perfect for what you are trying to test or experiment with. As a compute-centric lab, it's good for experiments with bare metal equipment, virtualization, containers, etc. However, it lacks many of the important features that would be needed for novel network configurations beyond simple VLAN separation.