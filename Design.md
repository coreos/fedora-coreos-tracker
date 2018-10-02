# Design Document

This document captures the design decisions made after discussion in issues. When a design issue is closed, the
conclusion should be summarized here with a link to the issue.

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

## Identification in `/etc/os-release`

Originally discussed in [#21](https://github.com/coreos/fedora-coreos-tracker/issues/21).

### Summary:

We will identify a Fedora CoreOS server using the `ID=fedora` and `VARIANT_ID=coreos`
fields in the `/etc/os-release` file.

## Cloud Agents

Originally discussed in [#12](https://github.com/coreos/fedora-coreos-tracker/issues/12).

### Summary:

 - FCOS will not ship cloud agents whenever possible.
 - Some clouds require the OS perform tasks like signaling boot completion. For those we will re-implement that functionality in
   Ignition or coreos-metadata.
 - For the short term, if we need to include an agent we will bake it into the image. We will not have any specific
   mechanism for including agents.

### Open questions:

 - What do we do about VMware, which has a very involved and intrusive "agent"?
