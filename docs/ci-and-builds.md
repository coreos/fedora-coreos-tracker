# CoreOS CI+build systems overview

Fedora CoreOS is tied/related to 3 major things:

 - Upstream git repositores like github.com/coreos/ignition, github.com/coreos/rpm-ostree, github.com/coreos/fedora-coreos-config, etc.
 - Actual releases of Fedora CoreOS via [the pipeline](https://github.com/coreos/fedora-coreos-pipeline)
 - Downstream [RHEL CoreOS](https://github.com/openshift/os)

## Infrastructure

- Github (specifically [the coreos namespace](https://github.com/coreos/))
- [quay.io](https://quay.io), specifically the [coreos-assembler](https://quay.io/coreos/coreos-assembler) namespace
- [CoreOS CI Jenkins](https://github.com/coreos/coreos-ci)
- [Fedora infrastructure](https://fedoraproject.org/wiki/Infrastructure)
- [OpenShift Prow](https://docs.ci.openshift.org/)

---

## Upstream CI

Most active repositories in the `coreos/` project are hooked up to at least one of 3 CI systems, being CoreOS CI Jenkins, Github Actions, or OpenShift Prow.  These 3 are the ones we are focusing on.

### CoreOS CI Jenkins

It is what we use on various repositories, and is how FCOS is released today via [the pipeline](https://github.com/coreos/fedora-coreos-pipeline).
We have a lot of institutional knowledge around this and it gives us a place where we can easily control the end-to-end interactions.  Jenkins is a well understood tool.

This is deployed in [CentOS CI](https://wiki.centos.org/QaWiki/CI) which is a bare metal OpenShift cluster where nested virt is enabled. 

Also of key relevance is the [coreos-ci-lib](https://github.com/coreos/coreos-ci-lib) repository.

### OpenShift Prow

Prow is heavily oriented towards testing OpenShift *container* components.  However, as of recently we enabled nested virt on the `build02` GCP cluster, which means we can create "container native" flows that still test the OS with [coreos-assembler](https://github.com/coreos/coreos-assembler/).

A specific reason to include Prow is that it contains tight integration with OpenShift which we need for RHCOS, and it is also maintained and staffed by a team that e.g. also contains a budget and secrets for running infrastructure in public clouds.

Examples can be found in the [openshift/release coreos/ folder](https://github.com/openshift/release/tree/main/ci-operator/config/coreos).

### GitHub Actions

Free for small scale, nice to use.  This is a good option for per-repository specific things that don't need centralization.

A good use case is e.g. validating rustfmt.

Examples:

 - https://github.com/coreos/rpm-ostree/blob/main/.github/workflows/rust-lints.yml

---

## quay.io/coreos-assembler namespace

A key aspect of Fedora CoreOS as well as RHEL CoreOS is [coreos-assembler](https://github.com/coreos/coreos-assembler).  As of today, we build it in quay.io and deliver it that way in the `quay.io/coreos-assembler` namespace.  The list of administrators for this namespace is managed independently of anything else.  If you think you need administrator access, file a ticket or ask on [`#coreos:fedoraproject.org` on Matrix](https://chat.fedoraproject.org/#/room/#coreos:fedoraproject.org).

### The buildroot container: quay.io/coreos-assembler/fcos-buildroot:testing-devel

Since [this pull request](https://github.com/coreos/fedora-coreos-config/pull/740), there is also a FCOS-oriented "buildroot" container that can be used in all CI systems.

## Fedora Infrastructure

Maintained by a distinct team.  FCOS and our container images include most content derived from Koji/Bodhi etc.

It would potentially make sense to have some of our containers built in Fedora too, such as coreos-assembler.  That would give us e.g. multi-arch.  But that is not being pursued currently.

