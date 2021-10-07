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

See https://docs.fedoraproject.org/en-US/fedora-coreos/platforms/

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

# Multipath

Multipath differs from other storage configurations by a major aspect: it is usually not
configured by Ignition. If we mount an individual path for e.g. `/sysroot`, multipathd will
not be able to take ownership afterwards. Furthermore, directly accessing individual paths
before `multipathd` takes over is unsafe (e.g. it could be a non-optimized path). And since
we need to mount `/boot` very early on, this naturally pushes multipath configuration into
kernel arguments (and ideally soon, initramfs overlays).

What we ended up with is adding an `rd.multipath=default` kernel argument which
triggers dracut to do "basic" automatic multipath setup in the stock initramfs:
https://github.com/dracutdevs/dracut/pull/780

By the nature of multipath, a tricky aspect is that e.g. the `by-label/root` symlink is
valid both *before* and *after* multipathd takes ownership. In order to safely wait for the
multipathed rootfs to show up, we have these udev rules which create, for example,
`by-label/dm-mpath-root`:

https://github.com/coreos/fedora-coreos-config/blob/94e0daa567a658f023d48ac5929c72ed910792bd/overlay.d/05core/usr/lib/udev/rules.d/90-coreos-device-mapper.rules#L1

This is why we require the `root=/dev/disk/by-label/dm-mpath-root` kernel argument; so that
the mount generated by `systemd-fstab-generator` waits for the the multipath version to show
up and doesn't just mount an individual path.

Firstboot (day-1) support is usually done at coreos-installer time by doing:

```
coreos-installer install \
  --append-karg rd.multipath=default \
  --append-karg root=/dev/disk/by-label/dm-mpath-root \
  --append-karg rw
  ...
```

The `rw` bit is necessary because `systemd-fstab-generator` will create a read-only mount by
default (usually, `rw` is injected by `rdcore rootmap` for subsequent boots, but this does
not happen if there is already a `root` karg).

That said, turning on multipath on a subsequent (day-2) boot is still supported if the
multipath setup itself is compatible with this. This is done by appending the same kargs as
above using e.g.  `rpm-ostree kargs`. (Appending the kargs can also be done via
`ignition-kargs`, though this still counts as "day-2" since on first boot we'd still access
the boot partition directly.)

We don't yet document multipath for FCOS, but we do document this setup for
OpenShift that has a kola test:

- https://github.com/coreos/coreos-assembler/blob/f5d003d2ebb81283c3e071ce2ac268884aa7232b/mantle/kola/tests/misc/multipath.go

We also support multipath on an individual non-root partition. See the test above for how
this works.

More links:

- https://github.com/coreos/ignition-dracut/issues/154
- https://bugzilla.redhat.com/show_bug.cgi?id=1944660
- https://github.com/coreos/fedora-coreos-config/pull/1011
