# Implementing a new emerging platform

This template is a simplified version of the
[full template](https://github.com/coreos/fedora-coreos-tracker/blob/main/.github/ISSUE_TEMPLATE/implementing-new-platform.md)
that only includes what is strictly needed to get initial support for a new
platform in Fedora CoreOS. This simplified version notably does not include the
steps needed to add new boot images to the release process.

Platforms added via this process are labelled "emerging" and users will have to
get boot images for them by converting existing images in the right format and
changing the `ignition.platform.id=<platform>` command line parameter.

This process will be documented using `guestfish` as an example.

## During Development

Create PRs addressing the following:

- [ ] [Ignition](https://github.com/coreos/ignition/) ([example PR](https://github.com/coreos/ignition/pull/918))
  - [ ] Add userdata fetch
  - [ ] If the platform supports it (unlikely), add userdata deletion
- [ ] [Afterburn](https://github.com/coreos/afterburn/) ([example PR](https://github.com/coreos/afterburn/pull/451))
  - [ ] (Cloud Only) Add relevant attributes
  - [ ] (Cloud Only) Add SSH key support if available
  - [ ] (Cloud Only) Add hostname support if available
  - [ ] (Cloud Only) Add check-in if needed (unlikely)
- [ ] [fedora-coreos-docs](https://github.com/coreos/fedora-coreos-docs) ([example PR](https://github.com/coreos/fedora-coreos-docs/pull/377))
  - [ ] Add a `provisioning-<platform>.adoc` that walks through how to setup the new platform
  - [ ] Add an entry in the `modules/ROOT/nav.adoc` that points to new documentation
- [ ] (Optional but recommended) Add support for the platform to [kola](https://github.com/coreos/coreos-assembler) to simplify testing
- Create or ask for a new upstream releases for:
  - [ ] Ignition
  - [ ] Afterburn
- Wait for new images with updated Ignition and Afterburn to reach stable then
  merge documentation with `guestfish` commands:
  - [ ] fedora-coreos-docs

## At Release

There are no "At Release" steps as we do not produce new boot images for
emerging platforms/
