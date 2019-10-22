# Release-index specifications

The release-index is a JSON document with a single object containing the following fields:

- `note` (optional, string): a human-friendly documentation text.
- `stream` (mandatory, string): name of the release stream.
- `metadata` (mandatory, object): metadata attributes for this JSON document.
  - `last-modified` (mandatory, string): UTC timestamp for the last change, in ISO 8601 format.
- `releases` (mandatory, list of objects): per-release details. Each entry MUST have a unique non-empty `version` field. The list MUST be sorted in ascending order, from oldest to latest release.
  - `version` (mandatory, string): release version identifier.
  - `metadata` (mandatory, string): URL to the release metadata document for this version.
  - `commits` (mandatory, list of objects): per-architecture OSTree commits. Each entry MUST have a unique non-empty `architecture` field.
    - `architecture` (mandatory, string): relevant base-architecture for this commit.
    - `checksum` (mandatory, string): OSTree commit identifier.
