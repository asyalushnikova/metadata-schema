# HCA Metadata release process specification

_This document is based on the specification initially publihsed as a Google doc [here](https://docs.google.com/document/d/1RBmq-AbKwqm2ElYXx2Iyxt54xyAsH5jn1stPzePmIao/edit#)._


## Overview

- [Objective](#objective)
- [Current process](#current-process)
- [Issues with current process](#issues-with-current-process)
- [Proposed updates](#proposed-updates)
- [Specification](#specification)
- [Assumptions](#assumptions)
- [Commitments](#commitments)
- [Issue handling](#issue-handling)


## Objective

This document is designed to define and specify the process by which HCA metadata is released from Github to schema.humancellatlas.org. This document does not cover the process by which a set of released metadata versions are allocated to an HCA data release.

## Current process

The current metadata release process is entirely based in Github, with the merge of metadata schema changes (files in the json-schema directory only, not documentation or code changes) from develop to master constituting the “release”. Any merges into master are picked up by a Git hook on schema.humancellatlas.org, which pulls in and deploys any updated or added metadata schema JSON files to the schema site.

## Issues with the current process

### Fully expanded ID and reference URLs in metadata entities

All HCA metadata schemas in Github currently has fully expanded ID and $ref URLs. This causes issues with validating and testing schema changes because validators, in particular out-of-the-box services, pull in dependent schemas from the URL rather than from the relative file path within the repo. This in turn causes continuous integration (Travis) builds in Github to fail until a schema change has been released, since changes in a dependent entity aren’t available on the new URL until after the release.

### Manual propagation of versioning changes

In the current process, it is necessary to manually propagate any version changes across all entities affected by a change. A minor change in a module entity for example would cause an equivalent minor change in any type entities referencing this module, as well as any bundle entities referencing the type. Due to the aforementioned testing issues, it is not possible to pick up a missed update until after the schema has been fully released. This makes the release process fragile and error prone.

## Proposed updates

In order to address the issues detailed above, we propose removing all fully expanded reference URLs and version numbers from the Github repo in favor of including relative URLs only. During a release, scripts will programmatically determine all relevant version number updates based on the type of metadata schema change and expand the relative URLs to fully expanded URLs. These scripts can be independent of actual merges from develop to master, so a merge to master no longer automatically constitutes a metadata release.

The new metadata schema update process will consist of the following steps:
1. Change to metadata schema in a local Git branch created from the latest develop branch
2. Update of version numbers in the version tracking file for any schemas affected by the change using a version number update script
3. Merging of local change and version number updates into develop
4. When one or more changes are ready to go to pre-release, merge develop into master
5. After appropriate commenting period, release metadata schema update xt

## Specification

### Version number updates

Given a metadata schema and an update type (patch, minor, or major), a script will identify the dependency tree for change and update the version number of all the relevant metadata entities in a central version tracking file. This script should be run for each individual metadata schema change.

### URL expansion

At release time, a script will use the updated version tracking file to expand the id and $ref URLs for all metadata entities to schema.humancellatlas.org. This script should only be used during the metadata release process.

### Release to schema.humancellatlas.org

Following URL expansion, all updated schemas should be copied to schema.humancellatlas.org. This constitutes the actual release.

## Testing

The Github Travis CI tests will be updated to use the local files for testing, so build failures will highlight actual JSON issues. After each release, a full DCP-wide integration test should be run using the new schemas to ensure that changes have successfully propagated through all DCP components.

Important to note here that merging from develop to master, which constitutes a pre-release, means that the updated metadata schema are available for DCP-wide testing prior to the official release. Other DCP components - data store, secondary pipelines, data portal - are able to test the new metadata schema within a predetermined amount of lead-time (based on the severity of the change) and report any issues.

## Assumptions

### Metadata updates vs release

- In order to avoid version churn, we will support grouping several patch-level metadata changes into one release, if this occurs. The downside of this approach is that from the users’ perspective, metadata schema version numbers might be skipped. For example, the donor_organism.json schema might upgrade from version 5.1.3 to version 5.1.6 in a single release because three individual patch updates were made to the schema since the previous release.
- Major (aka breaking) and minor version changes will always constitute a single release, although they may be grouped with one or more preceding patch releases.
- Major and minor releases will be announced ahead of time, with a defined release timeline (see below) and ideally after the changes have been made available in Github, in order to give other DCP members a chance to review changes and make any necessary changes to their resources.

## Commitments

- We commit to announce new schema versions and publish release notes via Slack #hca-metadata channel and metadata-wg@humancellcellatlas.org mailing list for every major, minor and patch release
- We commit to publishing pre-release versions for other DCP components to test for compatibility issues with major and minor schema changes. We will commit to giving 10 and 5 working days’ lead time for major and minor schema changes, respectively.
    - To work best, this requires metadata schema users to commit to paying attention to releases and flag their concerns if they need more review time.

## Issue handling

Schema updates will be checked for compatibility within the metadata-schema Github repository prior to pre-release, and will only be pre-released if all internal checks (i.e. Travis) pass. After a pre-release, schema updates will be checked for compatibility DCP-wide ideally through a Travis continuous integration test. Additionally, members of other DCP components should commit to specific checks if the schema updates are of high importance to them. After a pre-release is announced, we expect that some issues will be identified by other DCP components. Below, we outline how these issues will be addressed.

### Issue reported after pre-release, before release

- Issue: DCP component identifies a bug in the pre-released schema.
Response: Schema is patched, patch is recorded in pre-release notes, schema version is NOT incremented, <HOW DO WE HANDLE THE TIMEFRAME: RESET TIMER? START A NEW TIMER, MAYBE 1 WORKING DAY?>
- Issue: DCP component identifies an incompatibility in the pre-released schema AND the issue can be fixed by changing the schema.
Response: Schema is updated, update is recorded in pre-release notes, schema version is incremented only if the update supersedes the original version increment (e.g. if a minor change is required to fix an issue in a patch pre-release).
- Issue: DCP component identifies an incompatibility in the pre-released schema AND the issue cannot be fixed by changing the schema.
Response: Schema is not updated,

### Issue reported after release

- Issue: DCP component identifies a bug in the pre-released schema.
Response: Schema is updated (patch), updated schema goes through pre-release process.
- Issue: DCP component identifies an incompatibility in the released schema.
Response: Schema is updated (major or minor), updated schema goes through pre-release process.
