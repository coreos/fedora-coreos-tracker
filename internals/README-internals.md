# CoreOS Internals docs

This document intends to be a dumping ground to briefly describe various problem domains we've hit around building/delivering/testing CoreOS style systems.

Other important links:

 - https://github.com/coreos/coreos-assembler/
 - https://github.com/coreos/fedora-coreos-config

# Initramfs 

We use [dracut](https://github.com/dracutdevs/dracut/) the same as a number of other (but not all) distributions.  It basically gathers binaries/configuration from the real root and generates an initramfs from them.

The initramfs is critical to CoreOS systems; mainly https://github.com/coreos/ignition/ 
https://github.com/coreos/ignition-dracut/ handles running Ignition, and then the other key pieces are in the [fedora-coreos-config overlay.d](https://github.com/coreos/fedora-coreos-config), most notably `40ignition-ostree` which "glues together" the Ignition logic with the OSTree logic plus some CoreOS conventions.

The OSTree portion of the initramfs is reading the `ostree=` kernel command line argument to find the target root.  See [ostree-prepare-root.service](https://github.com/ostreedev/ostree/blob/d9fc1dd55d3ae0b71d303dceae9dd23d5b9497c8/src/boot/ostree-prepare-root.service).

A big recent effort is [reprovisioning the root filesystem](https://github.com/coreos/fedora-coreos-tracker/issues/94).

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
