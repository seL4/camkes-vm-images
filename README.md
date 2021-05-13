<!--
     Copyright 2018, Data61, CSIRO (ABN 41 687 119 230)

     SPDX-License-Identifier: CC-BY-SA-4.0
-->

# CAmkES VM Images

This repo contains a collection of binary images used in the CAmkES VMs. The aim
is to store binary blobs here and prevent them from bloating the repository sizes
of other source based repositories. Branches, tags, or specific revisions can be
used in `repo` manifests to refer to a particular version of this repo in a project
and then the binaries from that version can be accessed. The idea is that when
the repo size gets too large, we abandon the history and start again while tagging
the old tip.

## Licensing

For binaries with a GPL-like license, there should be information in this repository
that indicates where the sources can be found, including a tag or hash
to identify exactly which version is used (if the source reference is
to a git repository) and instructions for rebuilding (config files,
compiler used).  The aim is that anyone could grab the sources and
produce the same binary as is here.

For the license of any source files, see the [SPDX license tags][1] in their
header and the licence list in the [.reuse/dep5 file][2]. The directory
[`LICENSES`][3] contains the text for all licenses that are mentioned by files
in this repository.

[1]: https://spdx.org
[2]: .reuse/dep5
[3]: LICENSES/
