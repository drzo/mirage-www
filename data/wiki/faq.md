---
updated: 2013-12-09
author:
  name: Anil Madhavapeddy
  uri: http://anil.recoil.org
  email: anil@recoil.org
subject: Frequently Asked Questions (FAQ)
permalink: faq
---

*Q: Is MirageOS 'still' Linux? Does it require a Linux host on which to run?*

MirageOS is a '[library operating system](http://anil.recoil.org/papers/2013-asplos-mirage.pdf)', which means that it can run on any target for which a suitable bootloader and drivers exist. Versions earlier than 3 supported Unix and Xen targets; MirageOS version 3 added support for the KVM hypervisor via [solo5](/blog/introducing-solo5).  We have prototypes that compile the same application source code (e.g. the MirageOS website) to run as kernel modules inside FreeBSD, or even to JavaScript. MirageOS provides support to more easily target such diverse environments due to its emphasis on modular programming and compile-time specialisation.  If you'd like to dig into some code, see [the mirage-platform library](https://github.com/mirage/mirage-platform), which has the OCaml modules referred to in the Unix and Xen targets.

*Q: Is MirageOS 'production ready'?*

The 1.0 release is the first 'stable toolkit' release that is sufficient to self-host its infrastructure on the Internet.  [The current release](https://github.com/mirage/mirage/releases/latest) includes many improvements to stability and more supported features, including support for [gathering entropy](/blog/mirage-entropy), [execution trace collection and visualisation](http://roscidus.com/blog/blog/2014/10/27/visualising-an-asynchronous-monad/), and [ARM](/blog/introducing-xen-minios-arm).  The various libraries used in MirageOS (e.g. [mirage-tcpip](https://github.com/mirage/mirage-tcpip) and [Irmin](https://github.com/mirage/irmin)) follow their own release cadences, but similarly are in use in self-hosting infrastructure.  Some libraries are currently being used in large-scale commercial products like [Docker for Mac and Windows](https://blog.docker.com/2016/03/docker-for-mac-windows-beta/).

*Q: How does MirageOS compare against other cloud-friendly OS options (e.g like OSv) and also different approaches like like containers (e.g Docker)?*

MirageOS represents our desire for a radically simpler way of building complex distributed systems using a modern modular, functional and type-safe programming language such as OCaml. Unlike other cloud-friendly operating systems such as OSv, we do not attempt to optimize *existing* code, but instead focus on a toolkit to make it easier to quickly assemble *new* systems without having to be a domain expert in (e.g.) kernel programming.

The downside to our approach is that we only work with open protocols, since we cannot build clean-slate versions of closed protocols for which we have no specification.  Instead, we hope to build a system which enables easy development of arbitrary protocols and integration of them into the stack; a measure of our success is whether users can implement such protocols themselves, without specialised systems or kernel knowledge.

On the other hand, the current release contains clean-slate libraries for [TLS](https://github.com/mirleft/ocaml-tls), [TCP/IP](https://github.com/mirage/mirage-tcpip), [DNS](https://github.com/mirage/ocaml-dns), Xen [network](https://github.com/mirage/mirage-net-xen) and [storage](https://github.com/mirage/mirage-block-xen) device drivers], [HTTP](https://github.com/mirage/ocaml-cohttp), and other common Internet protocols, but all written in a completely type-safe fashion so that they are resistant to attacks such as buffer overflows that are plaguing the Internet. There's a good chance that a few years from now, existing systems will still be suffering those attacks, but MirageOS will continue to grow and mature its protocol implementations without sacrificing safety.

*Q: What's next for MirageOS?*

Back in the Cambridge Computer Laboratory, we are embarking on several major, multi-year projects that use MirageOS at their heart. The [User Centric Networking](http://usercentricnetworking.eu) project (with Nottingham and Technicolor among others) is building a privacy-preserving distributed system for recommender and content delivery systems. Instead of a monolithic cloud storing our personal data, we are porting MirageOS to Xen/ARM and deploying small, energy efficient devices inside people's homes. This "personal information hub" will talk to third-party service providers and enable control over what personal data is transmitted and allow users to balance their desire for social networking vs the cost of privacy breaches.

We've built prototypes of this technology in the past, but quickly discovered that securing and managing embedded devices running Linux and C code is incredibly difficult. Cloud providers employ an army of security professions to secure their perimeters, but embedded devices do not have this luxury. Since OCaml has supported fast native code compilation to ARM for over a decade, MirageOS provides the perfect balance of resource efficiency and security to drive these embedded systems and benefit the coming wave of the Internet of Things.

OCaml (the programming language that we use under the hood of MirageOS) also has deep connections to the formal methods community, with other major tools such as Coq (a widely used theorem prover) and CompCert (a verified C compiler) written in it. We have several initiatives ([http://rems.io](http://rems.io)) ongoing to verify components of MirageOS (such as the garbage collector), to support hardware compilation to FPGAs for datacenters (via the EPSRC-funded Networks-as-a-Service project) and support new experimental CPU targets such as the [BERI processor](http://www.cl.cam.ac.uk/research/security/ctsrd/beri.html).

We're extremely grateful to our research funding bodies (RCUK, EPSRC, EU FP7 and DARPA) for supporting such long-term research and making MirageOS possible. [Jane Street](http://janestreet.com) and [Citrix](http://www.citrix.com) have also contributed funding and expertise for an entire research group called [OCaml Labs](http://ocamllabs.io/) in the Cambridge Computer Lab to support the continued growth of the functional programming ecosystem. Anil has also recently published an O'Reilly book called Real World OCaml that's freely available at [https://realworldocaml.org](https://realworldocaml.org).

Last but not least it's simply more fun to exploit the flexibility of the MirageOS approach as a programmer to regain control over the myriad complexity of current software systems. Too much of modern systems construction involves wrestling with configuration files, mystical kernel policies, and occasionally documented APIs. Simply put, MirageOS is just a lot more enjoyable to use and develop code in when building server systems and network services. Several of the developers have replaced pieces of their personal infrastructure (like homepages, DNS servers, and home routers) with MirageOS unikernels!

