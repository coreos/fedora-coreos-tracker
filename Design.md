# Design Document

This document captures the design decisions made after discussion in issues. When a design issue is closed, the
conclusion should be summarized here with a link to the issue.

## Disk Layout

Originally discussed in issue [#18](https://github.com/coreos/fedora-coreos-tracker/issues/18). See also [dustymabe's comment](https://github.com/coreos/fedora-coreos-tracker/issues/18#issuecomment-409668929) summarizing the discussion in the FCOS meeting.

### Summary:

 - FCOS will use a "dd-able" image and ship with a standard partition layout.
 - The bare metal image and cloud images have the same layout.
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
    3	?	Root
    4	?	/var

We also want to support moving the root partition to new locations by recreating the OSTree at the new location. This
would involve downloading the OSTree repo contents and doing the deploy between the Ignition disks and files stage if
the root filesystem has changed. This is currently untested.

### Open Questions:

 - What do we do about 4k sector disks? We could make a "hybrid" disk image, but it technically breaks the GPT spec and
   may not work with poorly implemented UEFIs.
 - What is the exact partition layout?
 - Do we make /etc a ro bind mount?
 - What filesystem do we use for / and /var? ext4?
