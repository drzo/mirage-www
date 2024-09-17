---
updated: 2019-04-03
authors:
- name: Hannes Mehnert
  uri: https://github.com/hannesm
  email: hm519@cam.ac.uk
subject: MirageOS Spring 2019 hack retreat roundup
permalink: 2019-spring-retreat-roundup
---

Early March 2019, 31 MirageOS hackers gathered again in Marrakesh for our bi-annual hack retreat. We'd like to thank our amazing hosts, and everyone who participated on-site or remotely, and especially those who wrote up their experiences.
<img src="/graphics/spring2019.jpg" style="glot:right; padding: 15px" />

On this retreat, we ate our own dogfood, and used our MirageOS [DHCP](https://github.com/mirage/mirage-skeleton/tree/master/applications/dhcp), [recursive DNS resolver](https://github.com/robur-coop/unikernels/tree/master/resolver), and [CalDAV](https://github.com/robur-coop/caldav) unikernels as isolated virtual machines running on a [PC Engines APU](https://pcengines.ch/apu2c4.htm) with [FreeBSD](https://freebsd.org) as host system. The CalDAV server persisted its data in a git repository on the host system, using the raw git protocol for communication, the smart HTTP protocol could have been used as well. Lynxis wrote a [detailed blog post about our uplink situation](https://lunarius.fe80.eu/blog/mirageos-2019.html).

Lots of interesting discussions took place, code was developed, knowledge was exchanged, and issues were solved while we enjoyed the sun and the Moroccan food. The following list is not exhaustive, but gives an overview what was pushed forward.

## [Imagelib](https://github.com/rlepigre/ocaml-imagelib)

Imagelib is a library that can parse several image formats, and [eye-of-mirage](https://github.com/cfcs/eye-of-mirage) uses it to display those images in a [framebuffer](https://github.com/cfcs/mirage-framebuffer/).

During the retreat, imagelib was extended with [support for the BMP format](https://github.com/rlepigre/ocaml-imagelib/pull/22), it's [build system was revised to split off Unix-dependent functionality](https://github.com/rlepigre/ocaml-imagelib/pull/23), and preliminary support for the GIF format was implemented.

## [ActivityPub](https://github.com/kit-ty-kate/ocaml-activitypub)

ActivityPub is an open, decentralized social networking protocol, as used by [mastodon](https://mastodon.social). It provides a client/server API for creating, updating, and deleting content, and a federated server-to-server API for notifications and content delivery. During the retreat, an initial prototype of a protocol implementation was drafted.

## [opam](https://github.com/ocaml/opam)

Opam, the OCaml package manager, was extended in several directions:
- [External (OS package system) dependency integration](https://github.com/rjbou/opam/tree/depext)
- [Interleaving download with build/install actions](https://github.com/ocaml/opam/pull/3777)
- [Generalisation of the job scheduler](https://github.com/ocaml/opam/pull/3778)
- [JSON serialisation, including crowbar round-trip tests](https://github.com/ocaml/opam/pull/3776)
- Plugin evaluating (binary) [reproducibility](https://reproducible-builds.org/) of opam packages
- some smaller cleanup PRs ([return values](https://github.com/ocaml/opam/pull/3781), [locking code](https://github.com/ocaml/opam/pull/3783))

## [marracheck](https://github.com/Armael/marracheck/)

Work was started on a new utility to install as many opam packages as possible on a machine (there just wasn't enough choice with [opam-builder](https://github.com/OCamlPro/opam-builder), [opamcheck](https://github.com/damiendoligez/opamcheck) and [opam-check-all](https://github.com/kit-ty-kate/opam-check-all)). It uses opam-lib and Z3 to accomplish this.

## [Conex](https://github.com/hannesm/conex)

Conex is used for signing community repositories, esp. the opam-repository. Any opam package author can cryptographically sign their package releases, and users can verify that the downloaded tarball and build instructions are identical to what the author intended.

Conex has been developed since 2015, but is not yet widely deployed. We extended [opam-publish](https://github.com/ocaml/opam-publish) to invoke the `conex_targets` utility and sign before opening a pull request on the opam-repository.

## [SMTP](https://github.com/clecat/colombe)

The simple mail transfer protocol is an Internet standard for sending and receiving eMail. Our OCaml implementation has been improved, and it is possible to send eMails from OCaml code now.

## [HTTP2](https://github.com/anmonteiro/ocaml-h2)

The hypertext transfer protocol is an Internet standard widely used for browsing the world wide web. HTTP 1.1 is a line-based protocol which was specified 20 years ago. HTTP2 is an attempt to fix various shortcomings, and uses a binary protocol with multiplexing, priorities, etc. An OCaml implementation of HTTP2 has been actively worked on in Marrakesh.

## [Irmin](https://github.com/mirage/irmin)

Irmin is a distributed database that follows the same design principles as git. Soon, Irmin 2.0 will be released, which includes GraphQL, HTTP, chunk support, and can use the git protocol for interoperability. Irmin provides a key-value interface for MirageOS.

## OCaml compiler

Some hints on type errors for [int literals](https://github.com/ocaml/ocaml/pull/2301) and [int operators](https://github.com/ocaml/ocaml/pull/2307) were developed and merged to the OCaml compiler.

```
# 1.5 +. 2;;
         ^
Error: This expression has type int but an expression was expected of type
         float
       Hint: Did you mean `2.'?

# 1.5 + 2.;;
  ^^^ ^
Error: This expression has type float but an expression was expected of type
         int
Line 1, characters 4-5:
  Hint: Did you mean to use `+.'?
```

Also, the [whole program dead code elimination](https://github.com/ocaml/ocaml/pull/608) PR was rebased onto trunk.

## BGP / lazy trie

The [mrt-format](https://github.com/mor1/mrt-format) library which can parse multi-threaded routing toolkit traces, has been adapted to the modern OCaml ecosystem. The [border gateway protocol (BGP)](https://github.com/jimyuan1995/Mirage-BGP) library was slightly updated, one of its dependencies, [lazy-trie](https://github.com/mirage/ocaml-lazy-trie) was adapted to the modern ecosystem as well.

## Xen PVH

Xen provides several modes for virtualization.  MirageOS's first non-Unix target was the para-virtualized (PV) mode for Xen, which does not require hardware support from the hypervisor's host operating system but has a weak security profile (static mapping of addresses, large attack surface).  However, PV mode provides an attractive target for unikernels because it provides a simple software-based abstraction for dealing with drivers and a simple memory model; this is in contrast to hardware-virtualization mode, which provides greater security but requires more work from the guest OS.

A more modern virtualization mode combining the virtues of both approaches is PVH (formerly referred to as HVMLite), which is not yet supported by MirageOS.  Marek Marczykowski-Górecki from the [QubesOS](https://qubes-os.org) project visited to help us bring PVH support to the [unikraft project](https://xenproject.org/developers/teams/unikraft/), a common platform for building unikernels which we hope to use for MirageOS's Xen support in the future.

During the retreat, lots of bugs porting MirageOS to PVH were solved. It boots and crashes now!

## Learn OCaml as a unikernel

The platform learn OCaml embeds an editor, top-level, and exercises into a HTTP server, and allows students to learn OCaml, and submit solutions via the web interface, where an automated grader runs unit tests etc. to evaluate the submitted solutions. Teachers can assign mandatory exercises, and have an overview how the students are doing. Learn OCaml used to be executable only on a Unix host, but is now beeing ported into a MirageOS unikernel, executable as a standalone virtual machine.

## Network device driver (ixy)

The ixy network driver supports Intel 82599 network interface cards, and [is implemented in OCaml](https://github.com/ixy-languages/ixy.ml). Its performance has been improved, including several failing attempts which degraded its performance. Also, [it has been integrated into the mirage tool](https://github.com/mirage/mirage/pull/977) and is usable as a [mirage-net](https://github.com/mirage/mirage-net) implementation.

## DNS client API

Our proposed API is [described here](https://github.com/robur-coop/udns/blob/09c5e3c74c92505ec97f2a16818cc8a030e2868f/client/udns_client_flow.mli#L53-L80). Unix, Lwt, and MirageOS implementations are already available.

## [mirage-http](https://github.com/mirage/mirage-http) unified HTTP API

Since we now have two HTTP servers, [cohttp](https://github.com/mirage/ocaml-cohttp) and [httpaf](https://github.com/inhabitedtype/httpaf), in OCaml and MirageOS available, the new interface [mirage-http](https://github.com/mirage/mirage-http) provides a unified interface, and also supports connection upgrades to websockets.

## [cstruct](https://github.com/mirage/ocaml-cstruct) capabilities

We use cstruct, a wrapper around OCaml's [Bigarray](http://caml.inria.fr/pub/docs/manual-ocaml/libref/Bigarray.html), quite a lot in MirageOS. Until now, cstruct is a readable and writable byte array. We used phantom types to [add capabilities](https://github.com/mirage/ocaml-cstruct/pull/237) to the interface to distinct read-only and write-only buffers.

## [patch](https://github.com/hannesm/patch)

An OCaml implementation to apply unified diffs. This code has been extracted from conex (since we found some issues in it), and still needs to be fixed.

## Statistical memory profiler

Since 2016, Jacques-Henri Jourdan has been working on a [statistical memory profiler for OCaml](https://github.com/ocaml/ocaml/pull/847) (read the [OCaml 2016 paper](https://jhjourdan.mketjh.fr/pdf/jourdan2016statistically.pdf)). An [Emacs user interface](https://github.com/jhjourdan/statmemprof-emacs/) is available since some years. We integrated statmemprof into MirageOS unikernels [using the statmemprof-mirage library](https://github.com/hannesm/statmemprof-mirage), marshal the data via TCP, and provide a proxy that communicates with Emacs over a Unix domain socket, and the unikernel.

## [P2Pcollab](https://github.com/p2pcollab)

P2Pcollab is a collection of composable libraries implementing protocols for P2P collaboration.
So far various P2P gossip protocols has been implemented.
At this retreat the focus was on a gossip-based publish-subscribe dissemination protocol.
Future plans include building P2P unikernels and adding P2P pub/sub sync functionality to Irmin.

