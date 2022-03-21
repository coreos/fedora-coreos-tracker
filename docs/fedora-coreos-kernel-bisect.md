
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
- `fedora-5.16`
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
podman run -it --name=kbuild -v /path/to/kernel/git/:/path/to/kernel/git/ registry.fedoraproject.org/fedora:35
```

NOTE: try to use the same Fedora Cloud or Fedora container version as
      the version of Fedora you are targetting.

Once inside the VM or container we need to install some software to build the kernel:

```
sudo dnf update -y && \
sudo dnf install -y rpm-build rsync 'dnf-command(builddep)' && \
sudo dnf builddep -y kernel
# reboot here if in a VM
```

We can now make changes to the git repo (revert commits, etc) and run a few
commands to build the kernel. Before building we need to copy down the config
from the kernel dist-git repo and disable DEBUG symbols if they were enabled
(makes very large files): 

```
cd /path/to/kernel/git/
curl https://src.fedoraproject.org/rpms/kernel/raw/f35/f/kernel-x86_64-fedora.config > .config
sed -i 's/CONFIG_DEBUG_INFO=y/CONFIG_DEBUG_INFO=n/' .config
```

## 1. Directly Building and Installing the Kernel from Kernel Source git repo

To build and install the kernel directly on the system (i.e. on Fedora Cloud Base)
you can run the following:

```
make olddefconfig
make -j$(nproc) bzImage
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
make -j$(nproc) bzImage
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
Update the `manifest-lock.overrides.yaml` to specify the kernel and also update the manifest
to not specify `kernel-core` and `kernel-modules`. Here is an example:


```diff
diff --git a/manifest-lock.overrides.yaml b/manifest-lock.overrides.yaml
index 62cfbe5..81de60f 100644
--- a/manifest-lock.overrides.yaml
+++ b/manifest-lock.overrides.yaml
@@ -8,4 +8,6 @@
 # in the `metadata.reason` key, though it's acceptable to omit a `reason`
 # for FCOS-specific packages (ignition, afterburn, etc.).
 
-packages: {}
+packages:
+  kernel:
+    evr: 5.17.0_rc8+-2
diff --git a/manifests/bootable-rpm-ostree.yaml b/manifests/bootable-rpm-ostree.yaml
index 784acd4..734f374 100644
--- a/manifests/bootable-rpm-ostree.yaml
+++ b/manifests/bootable-rpm-ostree.yaml
@@ -7,7 +7,8 @@
 packages:
  # Kernel + systemd.  Note we explicitly specify kernel-{core,modules}
  # because otherwise depsolving could bring in kernel-debug.
- - kernel kernel-core kernel-modules systemd
+ - kernel systemd
  # linux-firmware now a recommends so let's explicitly include it
  # https://gitlab.com/cki-project/kernel-ark/-/commit/32271d0cd9bd52d386eb35497c4876a8f041f70b
  # https://src.fedoraproject.org/rpms/kernel/c/f55c3e9ed8605ff28cb9a922efbab1055947e213?branch=rawhide
```

After that you should be able to `cosa fetch --with-cosa-overrides && cosa build` like normal.


## Performing a Kernel Bisect

Now that we know how to build and use a kernel in various ways the bisect is
the easy part. Just follow the
[upstream kernel documentation](https://www.kernel.org/doc/html/latest/admin-guide/bug-bisect.html)
for doing a `git bisect` and repeat the build/test steps in between each step.
