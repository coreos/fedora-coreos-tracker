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

# Aleph version

`rpm-ostree status` will show admins the state of the ostree, but a few things live outside that and are not subject to in place updates.  For example, the on-disk filesystem (default `xfs`) and its specific layout, as well as the bootloader.

See [this pull request](https://github.com/coreos/coreos-assembler/pull/768/commits/2701e91838e18d3eac0694fd0a5f003befcfb218) which added `/sysroot/.coreos-aleph-version.json` that can be used to track the version of that data.

# ignition.platform.id

The design we have today is that each CoreOS system is the same OS content - the same OSTree commit,
and beyond that the exact same bootloader version, etc.

There are differences per platform on the image formats (VHD versus qcow2 vs raw, etc).  However,
what's *inside* the disk image for each platform is almost the same.

A key difference between each image is the `ignition.platform.id` kernel argument.  From the
moment the system boots and the kernel loads the initramfs, our userspace code uses this
to reliably know its target platform.  As could be guessed from the name, [https://github.com/coreos/ignition/](ignition)
uses this, and it runs early on.

But there's other code which dynamically dispatches on the platform ID:

- https://github.com/coreos/afterburn/
- [The time sync setup code](https://github.com/coreos/fedora-coreos-config/blob/d87b52bc6a90b53e1afeab2731b52612d5e3bbc0/tests/kola/chrony/coreos-platform-chrony-generator#L9)
- [network requirement detection](https://github.com/coreos/fedora-coreos-config/blob/d87b52bc6a90b53e1afeab2731b52612d5e3bbc0/overlay.d/05core/usr/lib/dracut/modules.d/35coreos-network/coreos-enable-network.service#L13)

Notice in particular how the time synchronization code ends up reconfiguring chrony dynamically.
For other operating systems which do "per cloud" disk images, it would have been more
natural to just change `/etc/chrony.conf` per platform.  But that would mean we have a different
ostree commit checksum per platform, breaking our "image based" update model.

It's very unlikely that we will change the platform IDs in the future.  However, FCOS users are recommended
to avoid parsing `ignition.platform.id`.  Generally, higher level code that needs to be
platform aware will have more platform-specific ways to find this information.

