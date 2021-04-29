# Rebase to a new version of Fedora (N)

## Release engineering changes

- [ ] Verify that a few tags have been created. These should have been created by releng scripts on branching: 

- `f${releasever}-coreos-signing-pending`
- `f${releasever}-coreos-continuous`

- [ ] The tag info for the coreos-pool tag has the new release (N) and next release (N+1) signing keys (just to stay ahead of the curve) and removes the old release (N-2) signing key. The following commands view the current settings and then update the list to 32/33/34 keys. You'll most likely have to get someone from releng to run the second command (`edit-tag`).

- `koji taginfo coreos-pool`
- `koji edit-tag coreos-pool -x tag2distrepo.keys="12c944d0 9570ff31 45719a39"`

- [ ] `koji untag` N-2 packages from the pool (at some point we'll have GC in place to do this for us, but for now we must remember to do this manually or otherwise distRepo will fail once the signed packages are GC'ed). For example the following snippet finds all RPMs signed by the Fedora 32 key and untags them.

Find the key short hash. Usually found [here](https://pagure.io/fedora-infra/ansible/blob/main/f/roles/bodhi2/backend/templates/pungi.rpm.conf.j2). Then:

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

Now we have a list of builds to untag. But we need one more sanity check. Let's make sure none of those are actually being used. Fire up the latest FCOS `testing-devel` and run:

```
f32key=12c944d0
key=$f32key
rpm -qai | grep -B 8 $key
```

If there are any RPMs signed by the old key they'll need to be investigated. Maybe they shouldn't be used any longer. Or maybe they're still needed.

After verifying the list looks good:

```
koji untag-build coreos-pool $untaglist
```

Now that untagging is done, give a heads up to rpm-ostree developers that N-2 packages have been untagged and that they may need to update their CI compose tests to freeze on a newer FCOS commit.

## coreos-installer changes

- [ ] Update coreos-installer to know about the signing key used for the future new major version of Fedora (N+1). Note that the signing keys for N+1 may not be created until releng branches and rawhide becomes N+1.

## Update [fedora-coreos-config](https://github.com/coreos/fedora-coreos-config/) `next-devel`

- [ ] Bump `releasever` in `manifest.yaml`
- [ ] Update the repos in `manifest.yaml` if needed
- [ ] Run `cosa fetch --update-lockfile`
- [ ] PR the result

## Ship rebased `next`

- [ ] Ship `next`

## Update [fedora-coreos-config](https://github.com/coreos/fedora-coreos-config/) `testing-devel`

- [ ] Bump `releasever` in `manifest.yaml`
- [ ] Update the repos in `manifest.yaml` if needed
- [ ] Run `cosa fetch --update-lockfile`
- [ ] Bump the base Fedora version in `ci/buildroot/Dockerfile`
- [ ] PR the result

## Ship rebased `testing`

- [ ] Ship `testing`

## Ship rebased `stable`

- [ ] Ship `stable`

## Miscellaneous container updates

- [ ] Rebase the coreos-assembler Dockerfile onto the new release
- [ ] Rebase the coreos-installer Dockerfile onto the new release
