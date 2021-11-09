# Rebase to a new version of Fedora (N)

## Release engineering changes

- [ ] Verify that a few tags have been created. These should have been created by releng scripts on branching: 

- `f${releasever}-coreos-signing-pending`
- `f${releasever}-coreos-continuous`

- [ ] The tag info for the coreos-pool tag has the new release (N) and next release (N+1) signing keys (just to stay ahead of the curve) and removes the old release (N-2) signing key. The following commands view the current settings and then update the list to 32/33/34 keys. You'll most likely have to get someone from releng to run the second command (`edit-tag`).

- `koji taginfo coreos-pool`
- `koji edit-tag coreos-pool -x tag2distrepo.keys="12c944d0 9570ff31 45719a39"`

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

## coreos-installer changes

- [ ] Update coreos-installer to know about the signing key used for the future new major version of Fedora (N+1). Note that the signing keys for N+1 may not be created until releng branches and rawhide becomes N+1. Drop the signing key for the obsolete stable release (N-1).

## Update [fedora-coreos-config](https://github.com/coreos/fedora-coreos-config/) `next-devel`

- [ ] Bump `releasever` in `manifest.yaml`
- [ ] Update the repos in `manifest.yaml` if needed
- [ ] Run `cosa fetch --update-lockfile`
- [ ] PR the result

## Ship rebased `next`

- [ ] Ship `next`
- Set a new update barrier for N-2 on `next`. In the barrier entry set a link to [the docs](https://docs.fedoraproject.org/en-US/fedora-coreos/update-barrier-signing-keys/). See [discussion](https://github.com/coreos/fedora-coreos-tracker/issues/480#issuecomment-631724629).

## Update [fedora-coreos-config](https://github.com/coreos/fedora-coreos-config/) `testing-devel`

- [ ] Bump `releasever` in `manifest.yaml`
- [ ] Update the repos in `manifest.yaml` if needed
- [ ] Run `cosa fetch --update-lockfile`
- [ ] Bump the base Fedora version in `ci/buildroot/Dockerfile`
- [ ] PR the result

## Ship rebased `testing`

- [ ] Ship `testing`
- Set a new update barrier for N-2 on `testing`. In the barrier entry set a link to [the docs](https://docs.fedoraproject.org/en-US/fedora-coreos/update-barrier-signing-keys/). See [discussion](https://github.com/coreos/fedora-coreos-tracker/issues/480#issuecomment-631724629).

## Disable `next-devel` stream

We prefer to disable `next-devel` when there is no difference between `testing-devel` and `next-devel`. This allows us to prevent wasting a bunch of resources (bandwidth, storage, compute) for no reason. After the switch to N if `next-devel` and `testing-devel` are in lockstep, then disable `next-devel`.

- [ ] Remove `next-devel` from the list of "development streams" in [the pipeline](https://github.com/coreos/fedora-coreos-pipeline/blob/main/streams.groovy). [Example PR.](https://github.com/coreos/fedora-coreos-pipeline/pull/343)
- [ ] Update the [promote-config job](https://github.com/coreos/fedora-coreos-streams/blob/main/.github/workflows/promote-config.yml) to promote `next` from `testing-devel`. [Example PR.](https://github.com/coreos/fedora-coreos-streams/pull/322)

## Ship rebased `stable`

- [ ] Ship `stable`
- Set a new update barrier for N-2 on `stable`. In the barrier entry set a link to [the docs](https://docs.fedoraproject.org/en-US/fedora-coreos/update-barrier-signing-keys/). See [discussion](https://github.com/coreos/fedora-coreos-tracker/issues/480#issuecomment-631724629).

## Miscellaneous container updates

These are various containers in use throughout our ecosystem. We should update or open a ticket to track updating them once a new Fedora release is out. If you open a ticket instead of doing the update add a link to the ticket as comment.

- [ ] Update coreos-assembler or open ticket to update:
    - [Dockerfile](https://github.com/coreos/coreos-assembler/blob/main/ci/Dockerfile)
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
