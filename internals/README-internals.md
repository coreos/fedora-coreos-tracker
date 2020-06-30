# CoreOS Internals docs

This document intends to be a dumping ground to briefly describe various problem domains we've hit around building/delivering/testing CoreOS style systems.

Other important links:

 - https://github.com/coreos/coreos-assembler/
 - https://github.com/coreos/fedora-coreos-config

# Initramfs 

This topic is big enough to have its own document: [README-initramfs.md](README-initramfs.md).

# CPU microcode

rpm-ostree runs dracut on the server side, and dracut knows how to pick up CPU microcode and prepend it to the initramfs.  Relevant bugs:

- https://bugzilla.redhat.com/show_bug.cgi?id=1199582
- https://bugzilla.redhat.com/show_bug.cgi?id=1803883

# Entropy

As of recently we enable `CONFIG_RANDOM_TRUST_CPU` which covers modern `x86_64` systems for example.

- https://bugzilla.redhat.com/show_bug.cgi?id=1830280
- https://github.com/openshift/machine-config-operator/issues/854

# Networking

In [this tracker issue](https://github.com/coreos/fedora-coreos-tracker/issues/24) a decision was made to use NetworkManager.  As of recently we use NetworkManager in the initramfs.  And even more recently, things have been reworked so that [afterburn can control initramfs networking](https://github.com/coreos/afterburn/pull/404) on specific clouds.

# Time synchronization

We use chrony, with some [additional custom logic for specific clouds](https://github.com/coreos/fedora-coreos-config/blob/faf387eac89d14924a1e2021d2093d0cdb8af8b3/overlay.d/20platform-chrony/usr/lib/systemd/system-generators/coreos-platform-chrony).
See also DHCP propagation: https://github.com/coreos/fedora-coreos-config/pull/412
