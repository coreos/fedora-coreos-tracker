
# Kernel regressions need bisecting

Sometimes we encounter kernel regressions and it is valuable to
identify the exact commit where a regression was introduced. An example
of this would be
[this issue for nodes booting in AWS](https://github.com/coreos/fedora-coreos-tracker/issues/1066#issuecomment-1019560658).

There are various strategies for how to determine the exact kernel
commit where a regression was introduced. Which strategy is most
efficient depends on the problem. Here they are:

1. directly building and installing the kernel from kernel source git repo
2. directly building and creating an RPM from the kernel source git repo

For `1.`, it only works if you can reproduce the problem on the
traditional `yum`/`dnf` based Fedora (like Fedora Cloud). If, however,
the problem only presents itself on Fedora CoreOS or is much easier to
reproduce on Fedora CoreOS (i.e. a `kola` test) then you'll want to
build the `rpm` (`2.`) and consume it that way.

## Kernel Source git Repos

There are a few kernel source git repositories to know about:

- `git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git`
    - Where the latest upstream development happens
- `git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/`
    - Where stable/LTS tags are handled (backports to stable branches happen here)
- `https://gitlab.com/cki-project/kernel-ark.git`
    - The `kernel-ark` repo where Red Hat patches/branches are maintained

The `kernel-ark` repo contains various branches used for feeding into
the [Fedora dist-git repo](https://src.fedoraproject.org/rpms/kernel). 
Here's a summary of what those branches are used for:


- `os-build`
    - The latest bits that track the under development yet to be release kernel.
- `fedora-6.3`
    - Follows a particular released kernel stream. This is where things are
      merged before they are fed into dist-git. If you want a commit reverted
      this is where it will land first.
- `ark-infa`
    - This branch contains all the Red Hat bits and nothing else. It can be merged
      on top of any other branch and then a SRPM can be created (`make dist-srpm`)
      for building using `rpmbuild --rebuild /path/to/srpm`.

## Creating a Kernel Build Environment

If running the kernel builds on a Fedora Cloud base machine where you
can install the kernel directly then you can set up the kernel build
environment directly in the VM. If not you'll probably want to use a
container for your kernel builds. Here's how to start up a container:

```
podman run -it --name=kbuild -v /path/to/kernel/git/:/path/to/kernel/git/ registry.fedoraproject.org/fedora:38
```

NOTE: try to use the same Fedora Cloud or Fedora container version as
      the version of Fedora you are targetting.

Once inside the VM or container we need to install some software to build the kernel:

```
sudo dnf update -y && \
sudo dnf install -y make rpm-build rsync 'dnf-command(builddep)' && \
sudo dnf builddep -y kernel
# reboot here if in a VM
```

We can now make changes to the git repo (revert commits, etc) and run a few
commands to build the kernel. Before building we need to copy down the config
from the kernel dist-git repo and disable making a DEBUG kernel if it was enabled,
which makes very large files:

```
cd /path/to/kernel/git/
RELEASE=f38 # or RELEASE=rawhide
curl "https://src.fedoraproject.org/rpms/kernel/raw/${RELEASE}/f/kernel-x86_64-fedora.config" > .config.fedora
cp .config.fedora .config
sed -i 's/CONFIG_DEBUG_KERNEL=y/CONFIG_DEBUG_KERNEL=n/' .config
```

## 1. Directly Building and Installing the Kernel from Kernel Source git repo

To build and install the kernel directly on the system (i.e. on Fedora Cloud Base)
you can run the following:

```
# Set make target. See https://src.fedoraproject.org/rpms/kernel/blob/rawhide/f/kernel.spec
make_target=bzImage # for x86_64 or vmlinux(ppc64le) or vmlinuz.efi(aarch64)
make olddefconfig
make -j$(nproc) $make_target
make -j$(nproc) modules
sudo make modules_install
sudo make install
```

On a Fedora Cloud base system the /boot partition is low on extra
space. In order to iterate (i.e. when running a `git bisect`) you can
restore the system back to it's old state before continuing. First,
modify the Makefile and set `EXTRAVERSION = bisect` and also
take a backup of the grub config:

```
sudo cp /boot/grub2/grub.cfg /boot/grub2/grub.cfg.bak
```

Then run the following script to build and install the kernel:

```
cat build.sh
#!/bin/bash
set -eux -o pipefail
cp .config.fedora .config
sed -i 's/CONFIG_DEBUG_KERNEL=y/CONFIG_DEBUG_KERNEL=n/' .config
# Set make target. See https://src.fedoraproject.org/rpms/kernel/blob/rawhide/f/kernel.spec
make_target=bzImage # for x86_64 or vmlinux(ppc64le) or vmlinuz.efi(aarch64)
make olddefconfig
make -j$(nproc) $make_target
make -j$(nproc) modules
sudo make modules_install
sudo make install
```

After testing and before running the next build I would restore and
free space with this clean script:

```
cat clean.sh
#!/bin/bash
set -eux -o pipefail
sudo cp /boot/grub2/grub.cfg.bak /boot/grub2/grub.cfg 
sudo rm -vf /boot/initramfs*bisect* /boot/vmlinuz-*bisect* /boot/System.map-*bisect*
sudo rm -rf /lib/modules/*bisect*
```

Then you can automate with:

```
bash clean.sh && bash build.sh
```

## 2. Directly Building and Creating an RPM from the Kernel Source git repo

In this scenario we're creating an RPM that can either then be package
layered on an existing FCOS system or used as input to a `cosa build`.

The commands here are:

```
make olddefconfig
make -j$(nproc) binrpm-pkg
```

### Package Layering the Kernel RPM

After copying the built kernel to the target machine you can install it with an override.
Example:

```
sudo rpm-ostree override replace ./kernel-5.17.0_rc8-1.x86_64.rpm --remove=kernel-core --remove=kernel-modules
```

### Doing a Build with COSA

Then copy the built RPM into the `overrides/rpm` folder under the COSA build directory.
After that you should be able to `cosa fetch --with-cosa-overrides && cosa build` like normal.

While iterating you should be able to skip the `cosa fetch` step. Just delete the old
RPM out of `overrides/rpm`, put the new one in place and then `cosa build`.


## Performing a Kernel Bisect

Now that we know how to build and use a kernel in various ways the bisect is
the easy part. Just follow the
[upstream kernel documentation](https://www.kernel.org/doc/html/latest/admin-guide/bug-bisect.html)
for doing a `git bisect` and repeat the build/test steps in between each step.

## Reporting issues upstream

Unfortunately the kernel doesn't have any git forge structure. It's
mostly email and mailing lists. If you want to report an issue
upstream you can run a command to give you what people/lists to email:

```
commit=abcdef
git format-patch --stdout "${commit}^..${commit}" | \
            ./scripts/get_maintainer.pl --norolestats
```

example:

```
$ commit=a09b314
$ git format-patch --stdout "${commit}^..${commit}" | ./scripts/get_maintainer.pl --norolestats
Jens Axboe <axboe@kernel.dk>
linux-block@vger.kernel.org
linux-kernel@vger.kernel.org
```

## Testing out fixes with Fedora's kernel

Once you have a proposed fix/patch you can easily build a Fedora kernel RPM by
adding your patch to the [`linux-kernel-test.patch` file](https://docs.fedoraproject.org/en-US/quick-docs/kernel/testing-patches/#_applying_the_patch)
in the [kernel distgit repo](https://src.fedoraproject.org/rpms/kernel).

After adding your patch you can then use `fedpkg` to build a new
kernel for your target architecture. For example:

```
fedpkg scratch-build --srpm --arch=x86_64
```

Once the build is complete you can grab the RPMs using the `koji` CLI:

```
koji download-task <task_id>
```

Placing these RPMs into the `overrides/rpm` directory and do a new COSA build
will give you a CoreOS build with the patched kernel.

After the tested patch looks good you can then open a PR to the `fedora-X.Y`
branch in the `kernel-ark` repo. See the above
[Kernel Source git Repos](#kernel-source-git-repos)
section for more details.
