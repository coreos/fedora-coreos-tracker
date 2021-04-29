# Rebase to a new version of Fedora (N)

## Release engineering changes

- [ ] Verify that a few tags have been created. These should have been created by releng scripts on branching: 

- `f${releasever}-coreos-signing-pending`
- `f${releasever}-coreos-continuous`

- [ ] The tag info for the coreos-pool tag has the new release (N) and next release (N+1) signing keys (just to stay ahead of the curve) and removes the old release (N-2) signing key. The following commands view the current settings and then update the list to 32/33/34 keys. You'll most likely have to get someone from releng to run the second command (`edit-tag`).

- `koji taginfo coreos-pool`
- `koji edit-tag coreos-pool -x tag2distrepo.keys="12c944d0 9570ff31 45719a39"`

- [ ] `koji untag` N-2 packages from the pool (at some point we'll have GC in place to do this for us, but for now we must remember to do this manually or otherwise distRepo will fail once the signed packages are GC'ed). For example the following snippet finds all RPMs signed by the Fedora 31 key and untags them.

```
f31key=3c3359c4
key=$f31key
untaglist=''
for build in $(koji list-tagged --quiet coreos-pool | cut -f1 -d' '); do
    if koji buildinfo $build | grep $key 1>/dev/null; then
        untaglist+="${build} "
        echo "Adding $build to untag list"
    fi
done

# After verifying the list looks good:
#   - koji untag-build coreos-pool $untaglist
```

## coreos-installer changes

- [ ] Update coreos-installer to know about the signing key used for the future new major version of Fedora (N+1). Note that the signing keys for N+1 may not be created until releng branches and rawhide becomes N+1.

## Update [fedora-coreos-config](https://github.com/coreos/fedora-coreos-config/) `next-devel`

- [ ] Bump `releasever` in `manifest.yaml`
- [ ] Update the repos in `manifest.yaml` if needed
- [ ] Run `cosa fetch --update-lockfile`
- [ ] PR the result

## Ship rebased `next`

- [ ] Ship `next`
- [ ] Set a new update barrier for N-2 on `next`. In the barrier entry set a link to [the docs](https://docs.fedoraproject.org/en-US/fedora-coreos/update-barrier-signing-keys/). See [discussion](https://github.com/coreos/fedora-coreos-tracker/issues/480#issuecomment-631724629).

## Update [fedora-coreos-config](https://github.com/coreos/fedora-coreos-config/) `testing-devel`

- [ ] Bump `releasever` in `manifest.yaml`
- [ ] Update the repos in `manifest.yaml` if needed
- [ ] Run `cosa fetch --update-lockfile`
- [ ] Bump the base Fedora version in `ci/buildroot/Dockerfile`
- [ ] PR the result

## Ship rebased `testing`

- [ ] Ship `testing`
- [ ] Set a new update barrier for N-2 on `testing`. In the barrier entry set a link to [the docs](https://docs.fedoraproject.org/en-US/fedora-coreos/update-barrier-signing-keys/).

## Ship rebased `stable`

- [ ] Ship `stable`
- [ ] Set a new update barrier for N-2 on `stable`. In the barrier entry set a link to [the docs](https://docs.fedoraproject.org/en-US/fedora-coreos/update-barrier-signing-keys/).

## Miscellaneous container updates

- [ ] Rebase the coreos-assembler Dockerfile onto the new release
- [ ] Rebase the coreos-installer Dockerfile onto the new release
