---
updated: 2019-04-26
authors:
- name: Thomas Leonard
  uri: http://roscidus.com/blog/
  email: talex5@gmail.com
subject: 'MirageOS security advisory 02: mirage-xen < 3.3.0'
permalink: MSA02
---

## MirageOS Security Advisory 02 - grant unshare vulnerability in mirage-xen

- Module:       mirage-xen
- Announced:    2019-04-25
- Credits:      Thomas Leonard, Mindy Preston
- Affects:      mirage-xen < 3.3.0,
                mirage-block-xen < 1.6.1,
                mirage-net-xen < 1.10.2,
                mirage-console < 2.4.2,
                ocaml-vchan < 4.0.2,
                ocaml-gnt (no longer supported)
- Corrected:    2019-04-22: mirage-xen 3.4.0,
                2019-04-05: mirage-block-xen 1.6.1,
                2019-04-02: mirage-net-xen 1.10.2,
                2019-03-27: mirage-console 2.4.2,
                2019-03-27: ocaml-vchan 4.0.2

For general information regarding MirageOS Security Advisories,
please visit [https://mirageos.org/security](https://mirageos.org/security).

### Background

MirageOS is a library operating system using cooperative multitasking, which can
be executed as a guest of the Xen hypervisor. Virtual machines running on a Xen
host can communicate by sharing pages of memory. For example, when a Mirage VM
wants to use a virtual network device provided by a Linux dom0:

1. The Mirage VM reserves some of its memory for this purpose and writes an entry
   to its *grant table* to say that dom0 should have access to it.
2. The Mirage VM tells dom0 (via XenStore) about the grant.
3. dom0 asks Xen to map the memory into its address space.

The Mirage VM and dom0 can now communicate using this shared memory.
When dom0 has finished with the memory:

1. dom0 tells Xen to unmap the memory from its address space.
2. dom0 tells the Mirage VM that it no longer needs the memory.
3. The Mirage VM removes the entry from its grant table.
4. The Mirage VM may reuse the memory for other purposes.

### Problem Description

Mirage removes the entry by calling the [gnttab_end_access][] function in Mini-OS.
This function checks whether the remote domain still has the memory mapped. If so,
it returns 0 to indicate that the entry cannot be removed yet. To make this function
available to OCaml code, the [stub_gntshr_end_access][] C stub in mirage-xen wrapped this
with the OCaml calling conventions. Unfortunately, it ignored the return code and reported
success in all cases.

### Impact

A malicious VM can tell a MirageOS unikernel that it has finished using some
shared memory while it is still mapped. The Mirage unikernel will think that
the unshare operation has succeeded and may reuse the memory, or allow it to be
garbage collected. The malicious VM will still have access to the memory.

In many cases (such as in the example above) the remote domain will be dom0,
which is already fully trusted. However, if a unikernel shares memory with an
untrusted domain then there is a problem.

### Workaround

No workaround is available.

### Solution

Returning the result from the C stub required changes to the OCaml grant API to
deal with the result. This turned out to be difficult because, for historical
reasons, the OCaml part of the API was in the ocaml-gnt package while the C stubs
were in mirage-xen, and because the C stubs are also shared with the Unix backend.

We instead created a [new grant API][] in mirage-xen, migrated all existing
Mirage drivers to use it, and then dropped support for the old API.
mirage-xen 3.3.0 added support for the new API and 3.4.0 removed support for the
old one.

The recommended way to upgrade is:
```bash
opam update
opam upgrade mirage-xen
```

### Correction details

The following PRs were part of the fix:

- [mirage-xen/pull/9](https://github.com/mirage/mirage-xen/pull/9) - Add grant-handling code to OS.Xen
- [mirage-net-xen/pull/85](https://github.com/mirage/mirage-net-xen/pull/85) - Use new OS.Xen API for grants
- [ocaml-vchan/pull/125](https://github.com/mirage/ocaml-vchan/pull/125) - Update to new OS.Xen grant API
- [mirage-block-xen/pull/79](https://github.com/mirage/mirage-block-xen/pull/79) - Port to new grant interface provided by mirage-xen
- [mirage-console/pull/75](https://github.com/mirage/mirage-console/pull/75) - Use new grant interface in mirage-xen
- [mirage-xen/pull/12](https://github.com/mirage/mirage-xen/pull/12) - Drop support for old ocaml-gnt package

### References

You can find the latest version of this advisory online at
[https://mirageos.org/blog/MSA02](https://mirageos.org/blog/MSA02).

This advisory is signed using OpenPGP, you can verify the signature
by downloading our public key from a keyserver (`gpg --recv-key
4A732D757C0EDA74`),
downloading the raw markdown source of this advisory from
[GitHub](https://raw.githubusercontent.com/mirage/mirage-www/master/tmpl/advisories/02.txt.asc)
and executing `gpg --verify 02.txt.asc`.

[gnttab_end_access]: https://github.com/mirage/mini-os/blob/94cb25eb73e58e5c825c1ad5f6cf3d2647603a50/gnttab.c#L98
[stub_gntshr_end_access]: https://github.com/mirage/mirage-xen/blob/v3.2.0/bindings/gnttab_stubs.c#L227
[new grant API]: https://github.com/mirage/mirage-xen/pull/9

