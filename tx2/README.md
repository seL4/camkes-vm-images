# TX2 port:

There is a bit of context to keep in mind when trying to handle this port:

* Linux makes calls down to some firmware within secure world, using the SMC
  opcode. The particular kconfig .config that we use happens to only generate
  these calls when trying to invalidate the BTB as part of the transient
  execution timing channel patches (spectre, meltdown, etc).

  ^ At some point, it may be necessary to emulate some subset of these SMC calls.
  ^ Some of them are sort of emulated by u-boot, and can be analyzed from there.

* The L4T source expects you to compile it using one of their prepackaged
  defconfigs so going off the trodden path is not impossible but you'll meet
  some curiosities.

  ^ These include actual compilation errors in the source, etc.

* The Linux binary and kconfig included here are for L4T, and *not* for
  the upstream Linux kernel.

  ^ At the time this port was initially done, Linux upstream was at version
    5.4.x.
  ^ The L4T binary and kconfig used here are for L4T based on a version 4.4
    kernel.
  ^ There was an L4T kernel based on upstream kernel 4.8, but the 4.4 version
    was chosen because it was based on the NVidia Jetpack version that claims
    to support the TX2.
    - Jetpack v4.1.1 claims not to support the TX2, and uses kernel 4.8.
    - Jetpack v3.3 claims to support the TX2, and uses kernel 4.4.
    - I did not bother to test to see whether or not Jetpack v4.1.1 can support
      the TX2, but if you feel like doing so in order to get the 4.8 kernel,
      then feel free to do so.
