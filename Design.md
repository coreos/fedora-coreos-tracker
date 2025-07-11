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
- [Configuration Language and Transpiler](#configuration-language-and-transpiler)
- [Security policies](#security-policies)
- [Bucket layout](#bucket-layout)
- [Version numbers](#version-numbers)

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
- `next`: The `next` stream represents the future. It will often be used to experiment with new features and also test out rebases of our platform on top of the next major version of Fedora. See [Major Fedora Version Rebases](#major-fedora-version-rebases) for more info.

All of these refs will be unversioned, in the sense that their names will not include the current Fedora major version.  The stream cadences are not contractual, but will initially have two weeks between releases.  The stream maintenance policies are also not contractual and may evolve from those described above, but changes will preserve the use cases and intended stability of each stream.

Users will be encouraged to run most of their production systems on `stable`, and a few percent of their systems on each of `next` and `testing` to catch regressions before they reach `stable`.

### Development Refs

Development for the next `testing` and `next` releases will occur in development refs.  These refs will be public, but will be stored in a different ostree repo from production refs.

- `testing-devel`: Nightly build of the package set that will be snapshotted for the next `testing` release.
- `next-devel`: Nightly build of the package set that will be snapshotted for the next `next` release.

### Mechanical Refs

There will also be some additional unversioned refs for the convenience of Fedora CoreOS developers.  These will be public and stored in the same ostree repo as development refs.  Unlike production and development refs, mechanical refs are not curated; they're simply a snapshot of the corresponding Bodhi repos, with no package pinning and no backports of fixes.  None of these refs are contractual; they might go away if we don't find them useful.

- `rawhide`: Nightly snapshot of rawhide.
- `branched`: Nightly snapshot of the upcoming Fedora release after it is branched.

### Out-of-Cycle Releases

Due to the promotion structure described above, `stable` can contain packages that are as much as four weeks out of date.  Sometimes, however, there will be an important bugfix or security fix that cannot wait a month to reach `stable` (or two weeks to reach `next` or `testing`).  In that case, the fix will be incorporated into out-of-cycle releases on affected streams.  These releases will not affect the regular promotion schedules; for example, a fix might sit in `testing` for only a few days before it is promoted to `stable`.

If a fix is important enough for an out-of-cycle `stable` release, other affected release streams should be updated as well.

In some cases it may make sense to apply a fix to `testing` but not issue an out-of-cycle release, allowing the fix to be picked up automatically when `testing` promotes to `stable`.

### Major Fedora Version Rebases

The release process integrates with Fedora's release milestones in the following ways:

- Fedora Beta Release
    - The `next` stream is switched over to the new release.
- Fedora Final Freeze
    - The `next` stream switches to weekly releases to closely track the GA content set.
- Fedora General Availability
    - Fedora CoreOS re-orients its release schedule in the following way:
        - Week -1 (Fedora "Go" Decision): `next` release:
          - `next` release with final Fedora GA content
        - Week 0 (GA release): triple release:
          - `stable` release promoted from previous `testing` (on N-1)
          - `testing` release promoted from previous `next`
          - `next` release contains latest Fedora N content, including Bodhi updates
        - Week 2: triple release:
          - `stable` release promoted from previous `testing`, now fully rebased to Fedora N
          - `testing` and `next` are now in sync

We have [a checklist](https://github.com/coreos/fedora-coreos-tracker/blob/main/.github/ISSUE_TEMPLATE/rebase.md) to track the exact steps followed during a rebase.

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

The partition layout will support "dual EFI/BIOS" on x86_64, and will have a single root partition as XFS by default.  We will support changing the root filesystem storage (but not `/boot`) via Ignition.

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

### Open questions:

 - What do we do about VMware, which has a very involved and intrusive "agent"?

## Supported Ignition Versions

Originally discussed in [#31](https://github.com/coreos/fedora-coreos-tracker/issues/31).

### Summary:

 - FCOS will only support Ignition spec 3.0.0 and up.
 - Ignition spec 3.0.0 will break compatibilty with spec 2.x.y, although most configs will only require minor changes.
 - Tooling should exist to aid converting 2.x.y configs to 3.0.0 configs, although perfect automated translation will not be possible.

## Configuration Language and Transpiler

- Originally discussed in issue [#129](https://github.com/coreos/fedora-coreos-tracker/issues/129).
- Versioning discussed in issue [#89](https://github.com/coreos/fedora-coreos-tracker/issues/89)

### Summary:

Fedora CoreOS will have a configuration language similar to the [Container Linux Configuration Language](https://coreos.com/os/docs/latest/configuration.html) named the Fedora CoreOS Configuration Language (FCCL). There will be a tool, the Fedora CoreOS Configuration Transpiler (FCCT) to convert Fedora CoreOS Configs (FCCs) to Ignition configs.

The FCCL will be versioned using semver, similar to how the Ignition spec is versioned. FCCT will accept all versions of the FCCL. Each FCCL version will target exactly one Ignition spec version.
This means:
- Old FCCs will continue to work with new versions of FCCT without modification.
- Each FCCL version will always emit the same version of Ignition config, regardless of what version of FCCT was used to transpile it.
- Since FCOS will accept old (down to 3.0.0) versions of Ignition configs, old FCCs will continue to work with new FCOS releases without modification.
- To use new features in new FCCT releases, users must update their configs to use the new FCCL spec.

## Security policies

### No autologin by default

Originally discussed in [#114](https://github.com/coreos/fedora-coreos-tracker/issues/114).

We will not enable autologin on serial or VGA consoles by default, even on platforms (e.g. Azure, DigitalOcean, GCP) which provide authenticated console access.  Doing so would provide an access vector that could surprise users unfamiliar with their platform's console access mechanism and access control policy.  For users who wish to use the console for debugging, we will provide documentation for using Ignition to enable autologin or to set a user password.

### Automatically disable SMT when needed to address vulnerabilities

Originally discussed in [#181](https://github.com/coreos/fedora-coreos-tracker/issues/181).

There have been multiple rounds of CPU vulnerabilities (L1TF and MDS) which cannot be completely mitigated without disabling Simultaneous Multi-Threading on affected processors. Disabling SMT has a cost: it reduces system performance and changes the apparent number of processors on the system. However, enabling SMT on affected systems would be an insecure default.

By default, Fedora CoreOS will configure the kernel to disable SMT on vulnerable machines. This conditional approach avoids incurring the performance cost on systems that aren't vulnerable. However, it fails to protect systems affected by undisclosed SMT vulnerabilities, and it allows future OS updates to disable SMT without notice if new vulnerabilities become known.

We will document this policy and its consequences, and provide instructions for unconditionally enabling or disabling SMT for users who prefer a different policy.

## Bucket Layout

Originally discussed in [#189](https://github.com/coreos/fedora-coreos-tracker/issues/189).

The `fcos-builds` bucket, fronted by http://builds.coreos.fedoraproject.org/ will be structured as follows:

```
/
  prod/
    streams/
      stable/
        releases.json
        builds/
          builds.json
          30.1234-5/
            release.json
            x86_64/
              meta.json
              commitmeta.json
              fedora-coreos-30.8-qemu.x86_64.qcow2.gz
              ostree-commit-object
              ostree-commit.tar
              ...
            ppc64le/
              ...
            ...
      testing/
      next/
      ...
  streams/
    stable.json
    testing.json
    ...
```

The artifacts under e.g. `30.1234-5/x86_64/` come directly from [coreos-assembler](https://github.com/coreos/coreos-assembler). The `/streams/*.json`, `release.json`, and `releases.json` are higher-level generated metadata objects. See [#98](https://github.com/coreos/fedora-coreos-tracker/issues/98) and [#207](https://github.com/coreos/fedora-coreos-tracker/pull/207) for more information about those.

The stream metadata format (under `/streams`) is intended to be stable, and stream metadata objects will contain links to artifacts in the release bucket. *Everything else about the bucket layout, including its directory structure and the formats of other metadata objects, is subject to change without notice. Third-party tooling should not rely on this structure, and should instead read metadata and artifact URLs directly from stream metadata at the officially documented URL*.

## Version numbers

Originally discussed in [#81](https://github.com/coreos/fedora-coreos-tracker/issues/81) and [#211](https://github.com/coreos/fedora-coreos-tracker/issues/211).

Fedora CoreOS versions will have the form `X.Y.Z.A`:

- X is the Fedora major version, e.g. `31`.
- Y is the datestamp that the package set was snapshotted from Fedora, e.g. `20191014`.  For mechanical streams, this is the build date.  For development and production streams, it's the date of the snapshot that was promoted.
- For official builds, Z is a code number corresponding to the stream:

Stream | Z version
-- | --
next | 1
testing | 2
stable | 3
next-devel | 10
testing-devel | 20
rawhide | 91
branched | 92

For developer builds (those not produced by the official pipeline), Z is always `dev`.

These Z codes were chosen to make production versions short and simple, development versions clearly related to production versions, and mechanical versions clearly separated into a distinct group.

- A is a revision number, which starts at 0 and is incremented for each new build with the same X.Y.Z parameters as an existing build.

Some examples:

Stream | Version | Comment
-- | -- | --
next | 32.20191018.1.0 | F32-based, first release from this snapshot
testing | 31.20191018.2.1 | F31-based, second release from this snapshot
stable | 31.20191001.3.1 | Second stable release from the 20191001 snapshot
next-devel | 31.20191018.10.10 | 11th build of the day
testing-devel | 31.20191018.20.0 |
rawhide | 33.20191018.91.0 | F33-based, first build of the day
branched | 32.20191018.92.0 |
(any developer build) | 31.20191018.dev.2 | Third build of the day

We are not committing to this version scheme indefinitely, and may change it in future if it proves unworkable.  A new Fedora major release (X bump) would be a good time to make such a change.  We don't intend Fedora CoreOS version numbers to be parsed by machine; they're meant to help humans quickly determine the salient properties of a release.
