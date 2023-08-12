
# Systemd regressions need bisecting

Similar to the kernel, systemd is often a core component of our
stack that has regressions that aren't easy to identify just by
inspecting a changelog.

## Systemd Source git Repos

There are a few kernel source git repositories to know about:

- `https://github.com/systemd/systemd.git`
    - Where the latest upstream development happens
- `https://github.com/systemd/systemd-stable.git`
    - Where stable/LTS tags are handled (backports to stable branches happen here)

There is also the [Fedora dist-git repo](https://src.fedoraproject.org/rpms/systemd). 

## Creating a Kernel Build Environment

You can use a container to build systemd from upstream.

```
SHARED=/path/to/shared/directory/
RELEASE=38
podman run -it --name=systemdbuild -v "${SHARED}:${SHARED}" "registry.fedoraproject.org/fedora:${RELEASE}"
```

```
sudo dnf update -y && \
sudo dnf install -y make rpm-build rsync 'dnf-command(builddep)' && \
sudo dnf builddep -y systemd
```

We can now make changes to the git repo (revert commits, etc) and run a few
commands to build systemd. If doing a
[`git` bisect](https://www.kernel.org/doc/html/latest/admin-guide/bug-bisect.html)
run the commands needed to start the bisect.

## Doing the systemd build/test

To build systemd you can run the following commands. These commands
were adapted from the notes in the
[Systemd README](https://github.com/systemd/systemd/blob/579fbe5b789cbee10546f6274c39be311e71e49c/README#L233-L247).


```
meson setup build/
```

And then the following can be iterated upon for each commit to test:

```
export DESTDIR=/path/to/shared/directory/fcos/overrides/rootfs/
ninja -C build && ninja -C build install
```

NOTE: If you run into `permission denied` errors when copying the files around check for SELinux denails.

Now you can run COSA to build/test. From the COSA directory:

```
cosa build && cosa kola run mytest
```

Now you can iterate until you find the problematic commit.
