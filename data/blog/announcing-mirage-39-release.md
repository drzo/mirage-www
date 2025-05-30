---
updated: 2020-10-27
authors:
- name: Martin Lucina
  uri: https://lucina.net/
  email: martin@lucina.net
- name: Hannes Mehnert
  uri: https://github.com/hannesm
  email: hm519@cam.ac.uk
subject: Announcing MirageOS 3.9.0
permalink: announcing-mirage-39-release
---

We are pleased to announce the release of MirageOS 3.9.0.

Our last release announcement was for [MirageOS 3.6.0](/blog/announcing-mirage-36-release), so we will also cover changes since 3.7.x and 3.8.x in this announcement.

New features:

- The Xen backend has been [re-written from scratch](https://github.com/mirage/mirage/issues/1159) to be based on Solo5, and now supports PVHv2 on Xen 4.10 or higher, and QubesOS 4.0.
- As part of this re-write, the existing Mini-OS based implementation has been retired, and all non-UNIX backends now use a unified OCaml runtime based on `ocaml-freestanding`.
- OCaml runtime settings settable via the `OCAMLRUNPARAM` environment variable are now exposed as unikernel boot parameters. For details, refer to [#1180](https://github.com/mirage/mirage/pull/1180).

Security posture improvements:

- With the move to a unified Solo5 and ocaml-freestanding base MirageOS unikernels on Xen gain several notable improvements to their overall security posture such as SSP for all C code, W^X, and malloc heap canaries. For details, refer to the mirage-xen 6.0.0 release [announcement](https://github.com/mirage/mirage-xen/releases/tag/v6.0.0).

API breaking changes:

- Several Xen-specific APIs have been removed or replaced, unikernels using these may need to be updated. For details, refer to the mirage-xen 6.0.0 release [announcement](https://github.com/mirage/mirage-xen/releases/tag/v6.0.0).

Other notable changes:

- `Mirage_runtime` provides event loop enter and exit hook registration ([#1010](https://github.com/mirage/mirage/pull/1010)).
- All MirageOS backends now behave similarly on a successful exit of the unikernel: they call `exit` with the return value 0, thus `at_exit` handlers are now executed ([#1011](https://github.com/mirage/mirage/pull/1011)).
- The unix backend used a toplevel exception handler, which has been removed. All backends now behave equally with respect to exceptions.
- Please note that the `Mirage_net.listen` function still installs an exception handler, which will be removed in a future release. The out of memory exception is no longer caught by `Mirage_net.listen` ([#1036](https://github.com/mirage/mirage/issues/1036)).
- To reduce the number of OPAM packages, the `mirage-*-lwt` packages are now deprecated. `Mirage_net` (and others) now use `Lwt.t` directly, and their `buffer` type is `Cstruct.t` ([#1004](https://github.com/mirage/mirage/issues/1004)).
- OPAM files generated by `mirage configure` now include opam build and installation instructions, and also an URL to the Git `origin` ([#1022](https://github.com/mirage/mirage/pull/1022)).

Known issues:

- `mirage configure` fails if the unikernel is under version control and no `origin` remote is present ([#1188](https://github.com/mirage/mirage/issues/1188)).
- The Xen backend has issues with event delivery if built with an Alpine Linux GCC toolchain. As a work-around, please use a Fedora or Debian based toolchain.

Acknowledgements:

- Thanks to Roger Pau Monné, Andrew Cooper and other core Xen developers for help with understanding the specifics of how Xen PVHv2 works, and how to write an implementation from scratch.
- Thanks to Marek Marczykowski-Górecki for help with the QubesOS specifics, and for forward-porting some missing parts of PVHv2 to QubesOS version of Xen.
- Thanks to @palainp on Github for help with testing on QubesOS.

