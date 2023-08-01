---
name: Implement a feature
about: Propose a design for a new feature
---

# Feature proposal

## Description

<!-- describe the concept and proposed implementation, in specific detail -->

## Implementation PRs

<!-- after PRs are posted, link them here, in a bulleted list so GitHub will show their status -->

## Did you consider?

<!-- please expand this list! -->

- Storage
  - [ ] Disk space usage
  - [ ] Behavior on 4Kn disks
  - [ ] Compatibility with multiple ESPs (Butane `boot_device.mirror`)
- First boot
  - [ ] Behavior on first boot vs. second boot
  - [ ] initrd networking requirements
  - [ ] Reprovisioned systems that reused existing storage devices
- OS update
  - [ ] Behavior after an OS rollback
  - [ ] Compatibility with old bootloaders
- Architectures
  - aarch64
    - [ ] Compatibility with non-UEFI boot
  - ppc64le
    - [ ] Whether new GRUB directives are supported by petitboot
  - s390x
    - [ ] Endianness issues
    - [ ] Need to rerun `zipl` to update kernel or kargs
    - [ ] ECKD/MBR lack of partition labels
    - [ ] ECKD maximum partition count
- Implementation
  - [ ] How interlocking PRs will be ratcheted into repos

## Implementation steps

- [ ] Create tracker ticket with initial design (above)
- [ ] Initial discussion and refinement in the ticket
- [ ] Add `meeting` label
- [ ] Discuss at community meeting
- [ ] Further refinement
  - [ ] Post draft [fedora-coreos-docs](https://github.com/coreos/fedora-coreos-docs/) PR, ideally before doing any implementation, to help identify design problems.
- [ ] Update issue description with final proposal and post a comment saying that you did
- [ ] Verify that rough consensus exists
- [ ] Implement.  Post PR links in the section above.  In the description of each PR, link to this issue and specify the prerequisites for merging.
  - [ ] Add kola test(s) for new feature
- [ ] Land implementation PRs, in order
- [ ] Wait for the functionality to reach FCOS stable
- [ ] Land docs PR
- [ ] Remove any ratcheting glue (e.g. workarounds in coreos-assembler)
