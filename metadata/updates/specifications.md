# Updates metadata specifications

The updates metadata is a JSON document with a single object containing the following fields:

- `stream` (mandatory, string): name of the update stream.
- `metadata` (mandatory, object): metadata attributes for this JSON document.
  - `last-modified` (mandatory, string): UTC timestamp for the last change, in ISO 8601 format.
- `releases` (mandatory, list of objects): per-release updates details. Each entry MUST have a unique non-empty `version` attribute.
  - `version` (mandatory, string): release version identifier.
  - `metadata` (mandatory, object): updates details.
    - `barrier` (optional, object): if present, the corresponding release is marked as an update barrier.
      - `reason` (mandatory, string): URL to a resource explaining the reason for this barrier.
    - `deadend` (optional, object): if present, the corresponding release is marked as an update dead-end.
      - `reason` (mandatory, string): URL to a resource explaining the reason for this dead-end.
    - `rollout` (optional, object): if present, the corresponding release is marked as an in-progress update rollout.
      - `start_epoch` (optional, signed integer): UNIX epoch timestamp for the start of this rollout. Default: `0`.
      - `start_percentage` (optional, float): percentage (ranging from `0.0` to `100.0`) for the starting point of this rollout. Default: `0.0`.
      - `duration_minutes` (optional, unsigned integer): duration in minutes for the rollout to progress till reaching 100% completion. Default: `0` (i.e. no progress).

# Glossary

- **barrier**: a release which is a forced chokepoint for auto-updates. Releases older than a certain barrier must first update to it before proceding to more recent updates.
- **dead-end**: a release which cannot further auto-update. Manual intervention is required to update out of it.
- **rollout**: a release which can be used as a valid target for auto-updates. Multiple rollouts can exist and progress in parallel.
