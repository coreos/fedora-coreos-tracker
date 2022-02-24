# Rebase to a new version of Fedora (N)

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
- [ ] Drop the signing key for the obsolete stable release (N-2).

### Update `rawhide` stream

- [ ] Update [manifest.yaml](https://github.com/coreos/fedora-coreos-config/blob/rawhide/manifest.yaml) to list N+1 as the releasever.

### Enable `branched` stream

- [ ] Update [manifest.yaml](https://github.com/coreos/fedora-coreos-config/blob/branched/manifest.yaml) to list N as the releasever.
- [ ] Update [streams.groovy](https://github.com/coreos/fedora-coreos-pipeline/blob/main/streams.groovy) to include the `branched` stream in the list of mechanical refs.


## At Fedora (N) Beta

### Update [fedora-coreos-config](https://github.com/coreos/fedora-coreos-config/) `next-devel`

- [ ] Bump `releasever` in `manifest.yaml`
- [ ] Update the repos in `manifest.yaml` if needed
- [ ] Run `cosa fetch --update-lockfile`
- [ ] PR the result

### Ship rebased `next`

- [ ] Ship `next`
- Set a new update barrier for N-2 on `next`. In the barrier entry set a link to [the docs](https://docs.fedoraproject.org/en-US/fedora-coreos/update-barrier-signing-keys/). See [discussion](https://github.com/coreos/fedora-coreos-tracker/issues/480#issuecomment-631724629).


## Preparing for Fedora (N) GA

### Update [fedora-coreos-config](https://github.com/coreos/fedora-coreos-config/) `testing-devel`

- [ ] Bump `releasever` in `manifest.yaml`
- [ ] Update the repos in `manifest.yaml` if needed
- [ ] Run `cosa fetch --update-lockfile`
- [ ] Bump the base Fedora version in `ci/buildroot/Dockerfile`
- [ ] PR the result


## At Fedora (N) GA

### Ship rebased `testing`

- [ ] Ship `testing`
- Set a new update barrier for N-2 on `testing`. In the barrier entry set a link to [the docs](https://docs.fedoraproject.org/en-US/fedora-coreos/update-barrier-signing-keys/). See [discussion](https://github.com/coreos/fedora-coreos-tracker/issues/480#issuecomment-631724629).

### Disable `branched` stream

- [ ] Update [streams.groovy](https://github.com/coreos/fedora-coreos-pipeline/blob/main/streams.groovy) to remove the `branched` stream in the list of mechanical refs.

### Untag old packages

`koji untag` N-2 packages from the pool (at some point we'll have GC in place to do this for us, but for now we must remember to do this manually or otherwise distRepo will fail once the signed packages are GC'ed). For example the following snippet finds all RPMs signed by the Fedora 32 key and untags them. Use this process:

- [ ] Find the key short hash. Usually found [here](https://pagure.io/fedora-infra/ansible/blob/main/f/roles/bodhi2/backend/templates/pungi.rpm.conf.j2). Then:

```
f32key=12c944d0
key=$f32key
untaglist=''
for build in $(koji list-tagged --quiet coreos-pool | cut -f1 -d' '); do
    if koji buildinfo $build | grep $key 1>/dev/null; then
        untaglist+="${build} "
        echo "Adding $build to untag list"
    fi
done
```

- [ ] Now we have a list of builds to untag. But we need one more sanity check. Let's make sure none of those are actually being used. Fire up the latest FCOS `testing-devel` and run:

```
f32key=12c944d0
key=$f32key
rpm -qai | grep -B 8 $key
```

If there are any RPMs signed by the old key they'll need to be investigated. Maybe they shouldn't be used any longer. Or maybe they're still needed.

- [ ] After verifying the list looks good:

```
koji untag-build coreos-pool $untaglist
```

- [ ] Now that untagging is done, give a heads up to rpm-ostree developers that N-2 packages have been untagged and that they may need to update their CI compose tests to freeze on a newer FCOS commit.

- [ ] Remove the N-2 signing key from the tag info for the coreos-pool tag. The following commands view the current settings and then update the list to the 33/34/35 keys. You'll most likely have to get someone from releng to run the second command (`edit-tag`).
    - `koji taginfo coreos-pool`
    - `koji edit-tag coreos-pool -x tag2distrepo.keys="9570ff31 45719a39 9867c58f"`


### Disable `next-devel` stream

We prefer to disable `next-devel` when there is no difference between `testing-devel` and `next-devel`. This allows us to prevent wasting a bunch of resources (bandwidth, storage, compute) for no reason. After the switch to N if `next-devel` and `testing-devel` are in lockstep, then disable `next-devel`.

- [ ] Follow the instructions [here](https://github.com/coreos/fedora-coreos-pipeline/tree/main/next-devel) to disable `next-devel`


## After Fedora (N) GA

### Ship rebased `stable`

- [ ] Ship `stable`
- Set a new update barrier for N-2 on `stable`. In the barrier entry set a link to [the docs](https://docs.fedoraproject.org/en-US/fedora-coreos/update-barrier-signing-keys/). See [discussion](https://github.com/coreos/fedora-coreos-tracker/issues/480#issuecomment-631724629).


## Miscellaneous container updates

These are various containers in use throughout our ecosystem. We should update or open a ticket to track updating them once a new Fedora release is out. If you open a ticket instead of doing the update add a link to the ticket as comment.

- [ ] Update coreos-assembler or open ticket to update:
    - [Dockerfile](https://github.com/coreos/coreos-assembler/blob/main/Dockerfile)
- [ ] Update coreos-installer
    - [Dockerfile](https://github.com/coreos/coreos-installer/blob/main/Dockerfile)
- [ ] Update Ignition
    - [Dockerfile.validate](https://github.com/coreos/ignition/blob/main/Dockerfile.validate)
- [ ] Update Butane
    - [Dockerfile](https://github.com/coreos/butane/blob/main/Dockerfile)
- [ ] Update fedora-coreos-cincinnati
    - [Dockerfile](https://github.com/coreos/fedora-coreos-cincinnati/blob/main/dist/fedora-infra/Dockerfile)
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
