# Fedora CoreOS metadata

Fedora CoreOS artifacts and streams are described by metadata objects, in the form of JSON documents.
This allows the general audience to consume releases and updates in a machine-friendly way.

The following types of metadata exist:
 * stream metadata
 * updates metadata
 * release index
 * release metadata

## Stream metadata

This document contains details about latest available artifacts, on each stream.

 * URL: `https://builds.coreos.fedoraproject.org/streams/${stream}.json`
 * Usage: consumed by the [getfedora.org download page](https://getfedora.org/en/coreos/download/)
 * (TODO) stream metadata JSON schema
 * [stream metadata sample][stream-sample]
 * [comments and rationale][stream-rationale]

[stream-sample]: ./stream/sample.json
[stream-rationale]: ./stream/rationale.yaml

## Updates metadata

This document contains details about updates and rollouts, on each stream.

 * URL: `https://builds.coreos.fedoraproject.org/updates/${stream}.json`
 * Usage: consumed by Cincinnati to discover valid update-paths
 * [JSON document specifications][updates-specs]
 * [updates metadata JSON schema][updates-schema]
 * [updates metadata sample][updates-sample]

[updates-schema]: ./updates/fcos-updates-schema.json
[updates-sample]: ./updates/sample.json
[updates-specs]: ./updates/specifications.md

## Release-index

This piece of metadata is meant to list all existing releases, on each stream.

 * URL: `https://builds.coreos.fedoraproject.org/prod/streams/${stream}/releases.json`
 * Usage: consumed by Cincinnati to discover valid releases
 * [JSON document specifications][release-index-specs]
 * [release-index JSON schema][release-index-schema]
 * [release-index sample][release-index-sample]

[release-index-schema]: ./release-index/fcos-release-index-schema.json
[release-index-sample]: ./release-index/sample.json
[release-index-specs]: ./release-index/specifications.md

## Release metadata

This document contains details about artifacts belonging to each release.

 * URL: dynamic for each release, provided by the release-index
 * Usage: internal tooling, artifacts mirroring, auditing
 * (TODO) release metadata JSON schema
 * [release metadata sample][release-sample]

[release-sample]: ./release/sample.json
