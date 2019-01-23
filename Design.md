# Design Document

This document captures the design decisions made after discussion in issues. When a design issue is closed, the
conclusion should be summarized here with a link to the issue.

- [OSTree Delivery Format](#ostree-delivery-format)
- [Release Streams](#release-streams)
- [Disk Layout](#disk-layout)
- [Approach towards shipping Python](#approach-towards-shipping-Python)
- [Identification in `/etc/os-release`](#identification-in-etcos-release)
- [Firewall management](#firewall-management)
- [Cloud Agents](#cloud-agents)
- [Supported Ignition Versions](#supported-ignition-versions)

## OSTree Delivery Format

- Originally discussed in issue [#23](https://github.com/coreos/fedora-coreos-tracker/issues/23). 

### Summary:

There are three proposed delivery models for delivering content to
end user systems:

- OSTree Repo: OSTree commits are stored in an OSTree repo (like a git
  repo) on a server and fetched via HTTP requests.
- rojig: uses a special rojig RPM and re-assembles OSTree commit from RPMs
  already on mirrors.
- OCI: OSTree commits are packaged up in OCI container images and delivered 
  via a container registry.

Currently the plan in Fedora CoreOS is to deliver content via a plain
OSTree Repo and augment our strategy with either rojig or OCI if it
proves useful or necessary.

## Release Streams

- Originally discussed in [#22](https://github.com/coreos/fedora-coreos-tracker/issues/22).

### Production Refs

Fedora CoreOS will have several refs for use on production machines.  At any given time, each ref will be downstream of a particular Fedora branch, and will consist of a snapshot of Fedora packages plus occasionally a backported fix.

- `testing`: Periodic snapshot of the current Fedora release plus Bodhi `updates`.
- `stable`: Promotion of a `testing` release, including any needed fixes.
- `next`:
  1. After Bodhi is enabled for the upcoming Fedora release, tracks that release; before then, tracks `testing`.
  2. After the upcoming kernel release has reached rc6 and before it goes final, tracks the rawhide kernel.  After the kernel goes final and before it is included in the tracked Fedora release, tracks the kernel from Bodhi `updates-testing`.

All of these refs will be unversioned, in the sense that their names will not include the current Fedora major version.  The stream cadences are not contractual, but will initially have two weeks between releases.  The stream maintenance policies are also not contractual and may evolve from those described above, but changes will preserve the use cases and intended stability of each stream.

Users will be encouraged to run most of their production systems on `stable`, and a few percent of their systems on each of `next` and `testing` to catch regressions before they reach `stable`.

### Development Refs

There will also be some additional unversioned refs for the convenience of Fedora CoreOS developers.  These will be public, but won't be exposed to users in the same way as production refs: they might be in a different repo, or in the same repo but not listed in the summary file.  None of these are contractual; they might go away if we don't find them useful.

- `rawhide`: Nightly snapshot of rawhide.
- `bodhi-updates`: Nightly snapshot of Bodhi `updates` for the Fedora release currently tracked by `testing`.
- `bodhi-updates-testing`: Nightly snapshot of Bodhi `updates-testing` for the Fedora release currently tracked by `testing`.

### Out-of-Cycle Releases

Due to the promotion structure described above, `stable` can contain packages that are as much as four weeks out of date.  Sometimes, however, there will be an important bugfix or security fix that cannot wait a month to reach `stable` (or two weeks to reach `next` or `testing`).  In that case, the fix will be incorporated into out-of-cycle releases on affected streams.  These releases will not affect the regular promotion schedules; for example, a fix might sit in `testing` for only a few days before it is promoted to `stable`.

A fix can take one of two forms:

1. An updated package taken directly from Fedora
2. A minimal fix applied to the package version already present in the affected stream

We'll need infrastructure for both approaches, and the ability to choose between them on a case-by-case basis.  Option 1 is cleaner and easier, but may not always be safe.  Option 2 is especially useful for the kernel, where we'll want to fix individual bugs without pushing an entire stable kernel update directly to the `stable` stream.

If a fix is important enough for an out-of-cycle `stable` release, other affected release streams should be updated as well.

In some cases it may make sense to apply a fix to `testing` but not issue an out-of-cycle release, allowing the fix to be picked up automatically when `testing` promotes to `stable`.

### Deprecation

Because production refs are unversioned, users will seamlessly upgrade between Fedora major releases, so compatibility must be maintained.  Removal of functionality will require explicitly announced deprecations, potentially with long deprecation windows.

## Disk Layout

- Originally discussed in issue [#18](https://github.com/coreos/fedora-coreos-tracker/issues/18). 
  See also [dustymabe's comment](https://github.com/coreos/fedora-coreos-tracker/issues/18#issuecomment-409668929)
  summarizing the discussion in the FCOS meeting.
- Filesystem details were discussed in [#33](https://github.com/coreos/fedora-coreos-tracker/issues/33).
  We will use XFS by default.

### Summary:

 - FCOS will use a "dd-able" image and ship with a standard partition layout.
 - The bare metal image and cloud images have the same layout.
 - The `/var` and root (`/`) filesystems will be XFS by default.
 - Anaconda will not be used for installation.
 - FCOS should not use the [GPT generator](https://www.freedesktop.org/software/systemd/man/systemd-gpt-auto-generator.html).
 - LVM should be supported, but not used by default.
 - Ignition will be used to customize disk layouts.

FCOS should have a fixed partition layout that Ignition can modify on first boot. The installer will be similar to the
Container Linux installer; the core of it will be dd'ing an image to the disk.

The partition layout is still undecided, but initial proposals look something like:

    Number	Type	Purpose
    -----------------------------------
    1	fat32	Boot partition/ESP
    2	N/A	Bios Boot partition
    3	XFS	Root
    4	XFS	/var

We also want to support moving the root partition to new locations by recreating the OSTree at the new location. This
would involve downloading the OSTree repo contents and doing the deploy between the Ignition disks and files stage if
the root filesystem has changed. This is currently untested.

### Open Questions:

 - What do we do about 4k sector disks? We could make a "hybrid" disk image, but it technically breaks the GPT spec and
   may not work with poorly implemented UEFIs.
 - What is the exact partition layout?
 - Do we make /etc a ro bind mount?

## Approach towards shipping Python

- Originally discussed in [#32](https://github.com/coreos/fedora-coreos-tracker/issues/32).

### Summary:

*TL;DR*

Fedora CoreOS group would really like to not ship python, but if we choose
that we want to keep a tool or a few tools in Fedora CoreOS that use python
then we should use an approach that makes python only available to the
operating system and not to end users.

**Note** that this does not say we will ship python.


*Details*

Container Linux has not shipped python in the past. Fedora is python
heavy and thus python has been shipped in the past in Fedora Atomic
Host. There are several reasons we've identified as reasons to not
ship python in Fedora CoreOS:

1. prevent users from running scripts directly on the host
2. prevent shipping/maintaining python
3. prevent issues where user's python script needs library X that isn't installed
4. prevent security issues in python requiring a respin
5. less space used on disk + less data transmitted for updates
6. better perception we're a minimal OS

Out of those we decided `#1` and `#3` were our primary concerns with
shipping python. For `#4` we determined there was not a significant
number of security issues to make shipping python prohibitive. We can
achieve the goals for `#1` and `#3` by shipping a *system python* that
is only accessible to operating system tools and not to end users.

## Identification in `/etc/os-release`

Originally discussed in [#21](https://github.com/coreos/fedora-coreos-tracker/issues/21).

### Summary:

We will identify a Fedora CoreOS server using the `ID=fedora` and `VARIANT_ID=coreos`
fields in the `/etc/os-release` file.

## Firewall management

Originally discussed in [#26](https://github.com/coreos/fedora-coreos-tracker/issues/26).

### Summary:

 - FCOS will ship without any ad-hoc filtering rules. By default, nodes will boot without firewall.
 - Components for both iptables and nft filtering will be provided (namely `iptables`, `nftables`, and `iptables-nft` packages, plus related kernel modules).
 - It will be possible to set up static rules (i.e. meant to be valid and unchanged for the whole node lifetime) via Ignition.
 - Dynamic rules (i.e. mutable at runtime) are out of scope for FCOS own toolings.
   Container runtimes and orchestrators take ownership of those via their own (containerized) rules managers.

## Cloud Agents

Originally discussed in [#12](https://github.com/coreos/fedora-coreos-tracker/issues/12).

### Summary:

 - FCOS will not ship cloud agents whenever possible.
 - Some clouds require the OS perform tasks like signaling boot completion. For those we will re-implement that functionality in
   Ignition or coreos-metadata.
 - For the short term, if we need to include an agent we will bake it into the image. We will not have any specific
   mechanism for including agents.

### AWS:

Originally discussed in [#66](https://github.com/coreos/fedora-coreos-tracker/issues/66).

- AWS does not require a cloud agent but does require NVME EBS udev rules
- The udev rules and script will be packaged in an RPM and included in FCOS with work being tracked in [#104](https://github.com/coreos/fedora-coreos-tracker/issues/104)

### Azure:

Originally discussed in [#65](https://github.com/coreos/fedora-coreos-tracker/issues/65).

- We've identified one major gap with not shipping the [Microsoft Azure Linux Agent](https://github.com/Azure/WALinuxAgent): the machine will not check-in and will eventually be culled by Azure for being stuck in the creation process.
- This gap will be covered by work done in [coreos-metadata](https://github.com/coreos/coreos-metadata/issues/120).
- One additional gap which will __not__ be covered is a lack of ephemeral disk support. We plan to ship udev rules but will not have a service which formats the disk unless we receive feature requests in the future. This was discussed in [#97](https://github.com/coreos/fedora-coreos-tracker/issues/97).
- As a cosmetic issue, we should also ship a rule to [ignore SR-IOV interfaces](https://github.com/coreos/fedora-coreos-tracker/issues/115).

### DigitalOcean:

Originally discussed in [#71](https://github.com/coreos/fedora-coreos-tracker/issues/71).

- DigitalOcean has an [agent](https://github.com/digitalocean/do-agent) that provides instance metrics back to DO.  We will not ship it.
- DigitalOcean does not generally offer DHCP.  Network configuration is obtained from an HTTP metadata service on a link-local address.  On other platforms this is handled by cloud-init.
- Networking should be configured by coreos-metadata running in the initramfs, but coreos-metadata [may need to learn to configure NetworkManager or nm-state](https://github.com/coreos/fedora-coreos-tracker/issues/111) depending on the outcome of [#24](https://github.com/coreos/fedora-coreos-tracker/issues/24).

### OpenStack:

Originally discussed in [#68](https://github.com/coreos/fedora-coreos-tracker/issues/68).

- OpenStack environments do not require a cloud agent
- We will provide any base level of functionality with ignition and coreos-metadata 

### Packet

Originally discussed in [#69](https://github.com/coreos/fedora-coreos-tracker/issues/69).

- The Packet cloud platform does not have an agent but does require instances to checkin via a [single HTTP POST](https://github.com/coreos/coreos-overlay/blob/1a846fbe914bed4583d02cc322ff011821d2fa8d/coreos-base/oem-packet/files/units/packet-phone-home.service). This will be [done via coreos-metadata](https://github.com/coreos/coreos-metadata/issues/120).
- For networking a machine's public IPv4 address is available via DHCP, but its private IPv4 and public IPv6 addresses are only available from the Packet metadata server, thus we will need to support configuring network using cloud metadata services [#111](https://github.com/coreos/fedora-coreos-tracker/issues/111).
- We will also want to supply support cloud-specific grub fragments [#110](https://github.com/coreos/fedora-coreos-tracker/issues/110) so that we can pass in a kernel `console=` argument that specifies `ttyS1`.

### Open questions:

 - What do we do about VMware, which has a very involved and intrusive "agent"?

## Supported Ignition Versions

Originally discussed in [#31](https://github.com/coreos/fedora-coreos-tracker/issues/31).

### Summary:

 - FCOS will only support Ignition spec 3.0.0 and up.
 - Ignition spec 3.0.0 will break compatibilty with spec 2.x.y, although most configs will only require minor changes.
 - Tooling should exist to aid converting 2.x.y configs to 3.0.0 configs, although perfect automated translation will not be possible.
