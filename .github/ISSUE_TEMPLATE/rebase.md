# Rebase to a new version of Fedora (N)

## At previous Fedora major release

### Open tickets to track related work for this release

- [ ] Fedora Changes Considerations ([example](https://github.com/coreos/fedora-coreos-tracker/issues/1222))
- [ ] Package Additions/Removals ([example](https://github.com/coreos/fedora-coreos-tracker/issues/1221))
- [ ] Test Week ([template](https://github.com/coreos/fedora-coreos-tracker/issues/new?template=test-week.md&title=tracker:+FN+Test+Week))
- [ ] Communications Tracker ([example](https://github.com/coreos/fedora-coreos-tracker/issues/1655))

## At Branching

Branching is when a new stream is "branched" off of `rawhide`. This eventually becomes the next major Fedora (N).

### Release engineering changes

- [ ] Verify that a few tags were created when branching occurred:

- `f${N+1}-coreos-signing-pending`
- `f${N+1}-coreos-continuous`

- [ ] Add and tag a package (any package) which is in the stable repos into the continuous tag. This will create the initial yum repo that's used as input for building the COSA container.

- `koji add-pkg --owner ${FAS_USERNAME} f${N+1}-coreos-continuous $PKG`
    - example: `koji add-pkg --owner dustymabe f36-coreos-continuous fedora-release`
    - This example uses the [`fedora-release`](https://src.fedoraproject.org/rpms/fedora-release) RPM, but it could be any other.
- `koji tag-build f${N+1}-coreos-continuous $BUILD`
    - example: `koji tag-build f36-coreos-continuous fedora-release-36-0.16`

- [ ] Add the N+1 signing key short hash (usually found [here](https://pagure.io/fedora-infra/ansible/blob/main/f/roles/bodhi2/backend/templates/pungi.rpm.conf.j2)) to the tag info for the coreos-pool tag. The following commands view the current settings and then update the list to the 32/33/34/35 keys. You'll most likely have to get someone from releng to run the second command (`edit-tag`).
    - `koji taginfo coreos-pool`
    - `koji edit-tag coreos-pool -x tag2distrepo.keys="12c944d0 9570ff31 45719a39 9867c58f"`

### coreos-installer changes

- [ ] Update coreos-installer to know about the signing key used for the future new major version of Fedora (N+1).
    - The current set of trusted signing keys is available at https://fedoraproject.org/security/.
    - If the Fedora (N+1) signing key isn't available yet at that site, you can also get it from https://src.fedoraproject.org/rpms/fedora-repos/tree/rawhide.
- [ ] Drop the signing key for the obsolete stable release (N-2).

Example PR: https://github.com/coreos/coreos-installer/pull/1113

### Update `rawhide` stream

- [ ] Update [manifest.yaml](https://github.com/coreos/fedora-coreos-config/blob/rawhide/manifest.yaml) to list N+1 as the releasever ([example PR](https://github.com/coreos/fedora-coreos-config/pull/2855))

### Enable `branched` stream

- [ ] Update [manifest.yaml](https://github.com/coreos/fedora-coreos-config/blob/branched/manifest.yaml) to list N as the releasever ([example PR](https://github.com/coreos/fedora-coreos-config/pull/2549))
- [ ] Update [config.yaml](https://github.com/coreos/fedora-coreos-pipeline/blob/main/config.yaml) to un-comment out the `branched` stream definition ([example PR](https://github.com/coreos/fedora-coreos-pipeline/pull/904))

## At Fedora (N) Beta

### Update [fedora-coreos-config](https://github.com/coreos/fedora-coreos-config/) `next-devel`

- [ ] Bump `releasever` in `manifest.yaml`
- [ ] Add the `fedora-candidate-compose` repo in `manifest.yaml` ([example PR](https://github.com/coreos/fedora-coreos-config/pull/2706))
- [ ] Update the repos in `manifest.yaml` if needed
- [ ] Run `cosa fetch --dry-run --update-lockfile`
    - this updates the x86_64 lockfile - the others will get updated when `bump-lockfile` runs.
    - in the future we may support [this](https://github.com/coreos/coreos-assembler/issues/3088) in `cosa fetch` directly
- [ ] PR the result

- [ ] Re-enable `next-devel` if needed ([docs](https://github.com/coreos/fedora-coreos-pipeline/tree/main/next-devel))
- [ ] Disable `branched` stream since it is no longer needed.
    - Update [config.yaml](https://github.com/coreos/fedora-coreos-pipeline/blob/main/config.yaml) to comment out the `branched` stream definition.

### Ship rebased `next`

- [ ] Ship `next`
- [ ] Set a new update barrier for the final release of N-1 on `next`. In the barrier entry set a link to [the docs](https://docs.fedoraproject.org/en-US/fedora-coreos/update-barrier-signing-keys/). See [discussion](https://github.com/coreos/fedora-coreos-tracker/issues/480#issuecomment-1247314065)


## Preparing for Fedora (N) GA

Do these steps as soon as we have a Go confirmation for GA, usually the Thursday of the week before GA.

### Ship a final `next` release

If the packages in `next-devel` don't exactly match the last `next` release that was done, we need to do a release with the final GA content. This ensures that what we'll promote to `testing` has the exact content in GA (plus version fast-tracks). This usually happens on the Thursday of the announcement of Go.

- [ ] Ensure final `next` release has GA content

### Build rebased `testing` and final `stable` release on N-1 

- [ ] Build `stable`; promote it from the `testing` branch, which should still be on N-1. Don't release it yet (i.e. don't run the `release` job).
- [ ] Build `testing`; promote it from the `next` branch instead of `testing-devel`. Don't release it yet (i.e. don't run the `release` job).

### Update [fedora-coreos-config](https://github.com/coreos/fedora-coreos-config/) `testing-devel`

- [ ] Bump `releasever` in `manifest.yaml`
- [ ] Update the repos in `manifest.yaml` if needed
- [ ] Sync the lockfiles for all arches from `next-devel`
- [ ] Bump the base Fedora version in `ci/buildroot/Dockerfile`
- [ ] PR the result


## At Fedora (N) GA

Do these steps on GA day.

### Release rebased `testing` and final `stable` release on N-1

- [ ] Run the `release` job for the staged `testing` and `stable` builds and start rollout.
- [ ] Set a new update barrier for the final release of N-1 on `testing`. In the barrier entry set a link to [the docs](https://docs.fedoraproject.org/en-US/fedora-coreos/update-barrier-signing-keys/). See [discussion](https://github.com/coreos/fedora-coreos-tracker/issues/480#issuecomment-1247314065)

### Disable `next-devel` stream if not needed

We prefer to disable `next-devel` when there is no difference between `testing-devel` and `next-devel`. This allows us to prevent wasting a bunch of resources (bandwidth, storage, compute) for no reason. After the switch to N if `next-devel` and `testing-devel` are in lockstep, then disable `next-devel`.

- [ ] Follow the instructions [here](https://github.com/coreos/fedora-coreos-pipeline/tree/main/next-devel) to disable `next-devel`

### Switch upstream packages to shipping release binaries from Fedora (N)

- [ ] Update [repo-templates](https://github.com/coreos/repo-templates) [config.yaml](https://github.com/coreos/repo-templates/blob/main/config.yaml) with the version number and GPG key ID for Fedora (N).

### Disable the `fedora-candidate-compose` repo

- [ ] Remove from the `manifest.yaml` of `next-devel` the `fedora-candidate-compose` repo

## After Fedora (N) GA

### Ship rebased `stable`

- [ ] Ship `stable`
- [ ] Set a new update barrier for the final release of N-1 on `stable`. In the barrier entry set a link to [the docs](https://docs.fedoraproject.org/en-US/fedora-coreos/update-barrier-signing-keys/). See [discussion](https://github.com/coreos/fedora-coreos-tracker/issues/480#issuecomment-1247314065)

### Update CoreOS Greenwave settings

- [ ] Open an MR against https://pagure.io/fedora-infra/ansible to remove the N-1 Fedora release from the version list in [fedora.yaml](https://pagure.io/fedora-infra/ansible/blob/c636c0581319b655cdc49079bd3793eb78abbe78/f/roles/openshift-apps/greenwave/templates/fedora.yaml#_528) (see https://pagure.io/fedora-infra/ansible/pull-request/2036 for context).

### Untag old packages

`koji untag` N-2 packages from the pool (at some point we'll have GC in place to do this for us, but for now we must remember to do this manually or otherwise distRepo will fail once the signed packages are GC'ed). For example the following snippet finds all RPMs signed by the Fedora 32 key and untags them. Use this process:

- [ ] Find the key short hash. Usually found [here](https://pagure.io/fedora-infra/ansible/blob/main/f/roles/bodhi2/backend/templates/pungi.rpm.conf.j2). Then:

```
f32key=12c944d0
key=$f32key
echo > untaglist # create or empty out file
for build in $(koji list-tagged --quiet coreos-pool | cut -f1 -d' '); do
    if koji buildinfo $build | grep $key 1>/dev/null; then
        echo "Adding $build to untag list"
        echo "${build}" >> untaglist
    fi
done
```

Now we have a list of builds to untag. But we need a few more sanity checks.

- [ ] Make sure none of the builds are used in `N` based FCOS. Check by running:

```
f32key=12c944d0
key=$f32key
podman run -it --rm quay.io/fedora/fedora-coreos:testing-devel rpm -qai | grep -B 9 $key
podman rmi quay.io/fedora/fedora-coreos:testing-devel
```

If there are any RPMs signed by the old key they'll need to be investigated. Maybe they shouldn't be used any longer. Or maybe they're still needed. One example of this is the shim RPM where the same build could be used for many Fedora releases. In this case you'll need to untag the RPM from `coreos-pool`, run a `koji distrepo`, which will remove that RPM from the repo metadata, and then re-tag it into the pool. The RPM in the repo will now be signed with a newer signing key.



- [ ] After verifying the list looks good, untag:

```
# use xargs so we don't exhaust bash string limit
cat untaglist | xargs -L50 koji untag-build -v coreos-pool
```

- [ ] Now that untagging is done, give a heads up to rpm-ostree developers that N-2 packages have been untagged and that they may need to update their CI compose tests to freeze on a newer FCOS commit.

- [ ] Remove the N-2 signing key from the tag info for the coreos-pool tag. The following commands view the current settings and then update the list to the 33/34/35 keys. You'll most likely have to get someone from releng to run the second command (`edit-tag`).
    - `koji taginfo coreos-pool`
    - `koji edit-tag coreos-pool -x tag2distrepo.keys="9570ff31 45719a39 9867c58f"`

### Open ticket for the next Fedora rebase

- [ ] Create a new ticket from the [rebase template](https://github.com/coreos/fedora-coreos-tracker/issues/new?assignees=&labels=area%2Fplatforms%2C+kind%2Fenhancement&template=rebase.md&title=tracker:+Rebase+onto+Fedora+N)
    - label with `FN` label where `N` is the Fedora version.


## Miscellaneous container updates

These are various containers in use throughout our ecosystem. We should update or open a ticket to track updating them once a new Fedora release is out. If you open a ticket instead of doing the update add a link to the ticket as comment.

- [ ] Update coreos-assembler or open ticket to update:
    - [Dockerfile](https://github.com/coreos/coreos-assembler/blob/main/Dockerfile)
    - [Dockerfiles for kola test containers](https://github.com/coreos/coreos-assembler/tree/main/tests/containers)
- [ ] Update coreos-installer
    - [Dockerfile](https://github.com/coreos/coreos-installer/blob/main/Dockerfile)
- [ ] Update Ignition
    - [Dockerfile.validate](https://github.com/coreos/ignition/blob/main/Dockerfile.validate)
- [ ] Update Butane
    - [Dockerfile](https://github.com/coreos/butane/blob/main/Dockerfile)
- [ ] Update fedora-coreos-cincinnati
    - [Dockerfile](https://github.com/coreos/fedora-coreos-cincinnati/blob/main/dist/fedora-infra/Dockerfile)
    - [ImageStream](https://pagure.io/fedora-infra/ansible/blob/main/f/roles/openshift-apps/coreos-cincinnati/templates/imagestream.yml)
    - [BuildConfig](https://pagure.io/fedora-infra/ansible/blob/main/f/roles/openshift-apps/coreos-cincinnati/templates/buildconfig.yml)
    - [Git Hash Variables (Optional)](https://pagure.io/fedora-infra/ansible/blob/main/f/roles/openshift-apps/coreos-cincinnati/vars)
- [ ] Update config-bot
    - [Dockerfile](https://github.com/coreos/fedora-coreos-releng-automation/blob/main/config-bot/Dockerfile)
- [ ] Update coreos-koji-tagger
    - [Dockerfile](https://github.com/coreos/fedora-coreos-releng-automation/blob/main/coreos-koji-tagger/Dockerfile)
    - [ImageStream](https://pagure.io/fedora-infra/ansible/blob/main/f/roles/openshift-apps/coreos-koji-tagger/templates/imagestream.yml)
    - [BuildConfig](https://pagure.io/fedora-infra/ansible/blob/main/f/roles/openshift-apps/coreos-koji-tagger/templates/buildconfig.yml)
- [ ] Update coreos-ostree-importer
    - [Dockerfile](https://github.com/coreos/fedora-coreos-releng-automation/blob/main/coreos-ostree-importer/Dockerfile)
    - [ImageStream](https://pagure.io/fedora-infra/ansible/blob/main/f/roles/openshift-apps/coreos-ostree-importer/templates/imagestream.yml)
    - [BuildConfig](https://pagure.io/fedora-infra/ansible/blob/main/f/roles/openshift-apps/coreos-ostree-importer/templates/buildconfig.yml)
- [ ] Update fedora-ostree-pruner
    - [Dockerfile](https://github.com/coreos/fedora-coreos-releng-automation/blob/main/fedora-ostree-pruner/Dockerfile)
    - [ImageStream](https://pagure.io/fedora-infra/ansible/blob/main/f/roles/openshift-apps/fedora-ostree-pruner/templates/imagestream.yml)
    - [BuildConfig](https://pagure.io/fedora-infra/ansible/blob/main/f/roles/openshift-apps/fedora-ostree-pruner/templates/buildconfig.yml)
