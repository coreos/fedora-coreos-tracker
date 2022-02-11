# The initramfs

For CoreOS the initramfs is critical; a key technological pillar of CoreOS is [Ignition](https://github.com/coreos/ignition/) which e.g. handles partitioning disks that happen on the first boot.  One way to think about this is that Ignition handles a lot of the roles that a traditional "installer" program might - our initramfs contains `sgdisk`, most others don't.

# Initramfs history

See the upstream Linux kernel document: ["what is initramfs"](https://www.kernel.org/doc/html/latest/filesystems/ramfs-rootfs-initramfs.html?highlight=initramfs#what-is-initramfs).

It's basically a small filesystem that gets passed to the kernel by the bootloader, and the kernel unpacks and runs it.

The high level goal of the initramfs is to mount the root filesystem (conventionally at `/sysroot`) and switch root into it, i.e. turning `/sysroot` into `/`.

# Initramfs technologies

We use [dracut](https://github.com/dracutdevs/dracut/) the same as a number of other (but not all) distributions.  It basically gathers binaries/configuration from the real root and generates an initramfs from them.

Modern systemd has a very clean design for both the initramfs and the real boot. See the ["man bootup"](https://www.freedesktop.org/software/systemd/man/bootup.html) documentation.  The software involved implements these abstract `.target` units.

There are 3 important pieces of software involved in the initramfs:
- [30ignition](https://github.com/coreos/ignition/tree/main/dracut/30ignition) (Part of Ignition)
- [ostree-prepare-root](https://github.com/ostreedev/ostree/blob/main/src/switchroot/ostree-prepare-root.c) (Part of OSTree)
- [40ignition-ostree dracut module](https://github.com/coreos/fedora-coreos-config/tree/testing-devel/overlay.d/05core/usr/lib/dracut/modules.d/40ignition-ostree) (fedora-coreos-config)

Note that Ignition and OSTree are both independent projects consumed by other distributions in addition to Fedora CoreOS.  This means that we want to support using each independently.  The `40ignition-ostree` dracut module *ties those two together* - it's the place where you will find systemd units that have direct ordering relationship around the two projects.

# First boot versus subsequent boot

Ignition runs only on the first boot.  To account for this, ignition-dracut ships two targets:

`ignition-complete.target`: Enabled on first boot  
`ignition-subsequent.target`: Enabled on every boot **except** the first

`-complete` will pull in a lot of units, such as `ignition-fetch.service` and `ignition-disks.service`

We implement`ignition-subsequent.target` today by hooking in `ignition-ostree-mount-subsequent-sysroot.service` which basically just waits for a filesystem with `LABEL=root` and mounts it - very simple!  But see below around the root filesystem.

# Images and finding filesystems

An important part of the CoreOS philosophy is to make bare metal as close to cloud workflows as possible.   This follows from moving all support for e.g. filesystem provisioning into Ignition.

[coreos-assembler](https://github.com/coreos/coreos-assembler) generates a disk image with `boot` (`/boot`) and `root` (`/`) labels.  Various components of the initramfs (as well as our default GRUB config) use the `label=boot` to find the boot partition.  The label `root` is used by `ignition-ostree-mount-firstboot-sysroot.service`.

# Live versus diskful

We also ship a "Live" ISO/PXE image which uses a different filesystem (squashfs).  This caused us to introduce a separate `ignition-diskful.target` which only runs on cases where we're booted from writable persistent storage (i.e. not ISO/PXE).

To implement the "live" or "run in RAM" aspects, the `live-generator` sets up an `overlayfs` for `/etc` and a `tmpfs` for `/var`.  Everything else is part of the `squashfs` which is read-only.

The Live OS setup differs currently between the ISO and PXE: https://github.com/coreos/fedora-coreos-tracker/issues/390

Currently when generating the ISO image we inject a label onto the root filesystem, and a `coreos.liveiso` kernel argument matching it.  The initramfs knows to look for that kernel argument, which it then uses to mount the squashfs which contains the root filesystem.

In contrast for PXE the squashfs is in the `live-initramfs` directly.

# /boot in the initramfs

There are multiple services which access the `/boot` partition in the initramfs. They are (in running order):
- `coreos-ignition-setup-user.service`: mounts `/boot` read-only to look for a user Ignition config. This is the first Ignition-related service to run.
- `coreos-copy-firstboot-network.service`: mounts `/boot` read-only to look for NetworkManager keyfiles. This unit runs after Ignition's `ignition-fetch-offline.service` but before networking is optionally brought up as part of `dracut-initqueue.service`.
- (on RHCOS) `rhcos-fips.service`: mounts `/boot` read-write to append `fips=1` to the BLS configs and reboot if FIPS mode is requested. This unit runs after `ignition-fetch.service` but before `ignition-disks.service`.
- `coreos-boot-edit.service`: mounts `/boot` read-write late in the initramfs process after `ignition-files.service` to make final edits (e.g. remove firstboot networking configuration files if necessary, append rootmap kargs to the BLS configs).

# SELinux in the initramfs

SELinux policy is loaded in the real root.  This means that every file we create in the initramfs must be relabeled.  See this code: https://github.com/coreos/fedora-coreos-config/blob/testing-devel/overlay.d/05core/usr/lib/dracut/modules.d/40ignition-ostree/coreos-relabel

# Networking

By default, the initramfs does not try to enable networking if it's not needed. This is important in the live ISO case. Software may request networking if they require it. For example, if Ignition detects a config which requires the network, it writes a stamp file at `/run/ignition/neednet` which we then detect and translate into `rd.neednet=1` via `coreos-enable-network.service`. For any other situation in which FCOS needs networking, we should add a triggering condition to that service. In the future if more cases are added, we may provide a cleaner API which does not require continuously expanding this list.

Network *enablement* is separate from network *configuration*. Afterburn handles rendering of network kernel arguments via [`afterburn-network-kargs.service`](https://github.com/coreos/afterburn/blob/e0c46db33ece0e003d278be73f2c83e237b315d0/dracut/30afterburn/afterburn-network-kargs.service). On some platforms, it may use a backchannel to fetch the network kargs. By default, it will use `AFTERBURN_NETWORK_KARGS_DEFAULT`, which is defined in [the fedora-coreos-config repo](https://github.com/coreos/fedora-coreos-config/blob/82f22f92620b60b009e94872a7b44fade8e782e1/overlay.d/05core/usr/lib/dracut/modules.d/35coreos-network/50-afterburn-network-kargs-default.conf) to be `ip=auto`.

For more details of the design, see https://github.com/coreos/fedora-coreos-tracker/issues/460 as well as the project [documentation](https://docs.fedoraproject.org/en-US/fedora-coreos/sysconfig-network-configuration/).

# Reprovisioning the root

A big recent effort is [reprovisioning the root filesystem](https://github.com/coreos/fedora-coreos-tracker/issues/94).  This will make the "subsequent" boot path work differently based on configuration.

# Debugging the initramfs

- https://fedoraproject.org/wiki/How_to_debug_Dracut_problems is generally useful
- Use `cosa buildinitramfs-fast` for fast iteration: https://github.com/coreos/coreos-assembler/pull/1433
- ignition-dracut contains code to dump the journal to a virtio channel: https://github.com/coreos/ignition-dracut/pull/146  - This is used by parts of coreos-assembler
