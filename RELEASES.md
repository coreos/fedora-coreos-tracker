# Fedora CoreOS Releases

FCOS releases normally happen every 2 weeks. In addition,
there may be asynchronous releases for e.g. CVEs or bug
fixes.

For more details, see
[the design doc](Design.md#release-streams).

## Streams

There are 3 primary streams: `stable`, `testing`, and
`next`. The `next` stream either tracks `testing` or the
next major version of Fedora. Content in `testing` is
promoted to `stable` after 2 weeks.

For more details, see
[the design doc](Design.md#release-streams).

## Versioning

FCOS versions are of the form `X.Y.Z.A`, where `X` is the
Fedora major version, `Y` is the date of the Fedora RPM
snapshot, `Z` is a stream identifier, and `A` is a revision
number.

Since `stable` releases are promoted from `testing`, their
`Y` dates will usually be 2 weeks behind `testing`.

For more details, see
[the design doc](Design.md#version-numbers).

## Schedule

The release schedule and release owners are tracked in a
[HackMD document](https://hackmd.io/WCA8XqAoRvafnja01JG_YA).
