<!--
     Copyright 2018, Data61
     Commonwealth Scientific and Industrial Research Organisation (CSIRO)
     ABN 41 687 119 230.

     This software may be distributed and modified according to the terms of
     the BSD 2-Clause license. Note that NO WARRANTY is provided.
     See "LICENSE_BSD2.txt" for details.

     @TAG(DATA61_BSD)
-->
# camkes-vm-images

This repo contains a collection of binary images used in the Camkes VMs. The aim
is to store binary blobs here and prevent them from bloating the repository sizes
of other source based repositories. Branches, tags, or specific revisions can be
used in `repo` manifests to refer to a paticular version of this repo in a project
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
