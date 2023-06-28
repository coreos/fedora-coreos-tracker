# Implementing a new supported platform

## During Development

Create PR's addressing the following:

- [ ] [Ignition](https://github.com/coreos/ignition/) ([example PR](https://github.com/coreos/ignition/pull/918))
  - [ ] Add userdata fetch
  - [ ] If the platform supports it (unlikely), add userdata deletion
- [ ] [Afterburn](https://github.com/coreos/afterburn/) ([example PR](https://github.com/coreos/afterburn/pull/451))
  - [ ] (Cloud Only) Add relevant attributes
  - [ ] (Cloud Only) Add SSH key support if available
  - [ ] (Cloud Only) Add hostname support if available
  - [ ] (Cloud Only) Add check-in if needed (unlikely)
- [ ] [stream-metadata-go](https://github.com/coreos/stream-metadata-go) ([example PR](https://github.com/coreos/stream-metadata-go/pull/45/))
  - [ ] Add platform to the `Media` struct in `release/release.go`
  - [ ] Add supporting code for new platform to `toStreamArch` func in `release/translate.go`
  - [ ] (Cloud Only) Cloud Images need to have an `Images` field
- [ ] (Cloud Only) [stream-metadata-rust](https://github.com/coreos/stream-metadata-rust/) ([example PR](https://github.com/coreos/stream-metadata-rust/pull/16))
- [ ] [fedora-coreos-tracker](https://github.com/coreos/fedora-coreos-tracker/) ([example PR](https://github.com/coreos/fedora-coreos-tracker/pull/1213))
  - [ ] Update the metadata for the new platform
- [ ] [coreos-assembler](https://github.com/coreos/coreos-assembler) ([example PR](https://github.com/coreos/coreos-assembler/pull/2489))
  - [ ] Implement required functionality to support new platform
- [ ] [fedora-coreos-config](https://github.com/coreos/fedora-coreos-config/)
  - [ ] Add a stanza to `platforms.yaml` if the system should use a serial console, or both serial and graphical consoles
- [ ] [fedora-websites-3.0](https://gitlab.com/fedora/websites-apps/fedora-websites/fedora-websites-3.0/)
  - [ ] Add friendly name for platform to `components/utilities/FpDownloadItem.vue`
  - [ ] Add artifact to `pages/coreos/download.vue`
  - [ ] Possibly add logo to `content/editions/coreos/home.yml`
- [ ] [fedora-coreos-browser](https://github.com/coreos/fedora-coreos-browser) ([example PR](https://github.com/coreos/fedora-coreos-browser/pull/35))
  - [ ] Add a list element for new platform in `browser/index.html`
- [ ] [build pipeline](https://github.com/coreos/fedora-coreos-pipeline) ([example PR](https://github.com/coreos/fedora-coreos-pipeline/pull/815))
  - [ ] Add platform to the list found in `config.yaml` for building the new artifact
- [ ] [fedora-coreos-docs](https://github.com/coreos/fedora-coreos-docs) ([example PR](https://github.com/coreos/fedora-coreos-docs/pull/377))
  - [ ] Add a `provisioning-<platform>.adoc` that walks through how to setup the new platform
  - [ ] Add an entry in the `modules/ROOT/nav.adoc` that points to new documentation

## At Release

1. Merge upstream changes and put out a release:
   - [ ] Ignition
   - [ ] Afterburn
1. Merge metadata changes:
   - [ ] stream-metadata-go
   - [ ] stream-metadata-rust
   - [ ] fedora-coreos-tracker
   - [ ] fedora website
   - [ ] fedora-coreos-browser
1. Create and push signed tags with appropriate versions
   ```
   # Ensure gpg key for signing in github settings that is associated to redhat email.
   # Verify you are on the upstream repo's main branch.

   git status

   RELEASE_VER=vx.y.z
   # Replace 'x.y.z' with the appropriate numbers.

   git tag -s ${RELEASE_VER}
   # Give appropriate detail to tag, check previous tags with 'git show ${RELEASE_VER}'

   git push git@github.com:coreos/targeted-repo.git ${RELEASE_VER}
   # Navigate to the targeted-repo's tag section to ensure a valid signed tag is listed.
   # e.g. https://github.com/...repo/tags
   ```
   1. [ ] Tag stream-metadata-go following the above steps. After tagging, ensure that dependabot has picked up latest version, and merged it into fedora-coreos-stream-generator && coreos-assembler.
      - These can be triggered manually by navigating to [fedora-coreos-stream-generator's Dependabot](https://github.com/coreos/fedora-coreos-stream-generator/network/updates/) and [coreos-assembler's Dependabot](https://github.com/coreos/coreos-assembler/network/updates) respectively; then, clicking "Check for updates".
        - This might need to be done a few times, as the Dependabot might not pickup tag changes for a few attempts after initial tagging.
   2. [ ] Tag fedora-coreos-stream-generator following the above steps.
1. Merge the following changes:
   - [ ] coreos-assembler
1. Wait for updates made to coreos-assembler to be propagated to latest container
   - [ ] Download latest version of coreos-assembler container. Verify platform support functionality.
1. Merge changes for:
   - [ ] Build pipeline
1. Wait for new images to reach stable then merge documentation.
   - [ ] fedora-coreos-docs merged
