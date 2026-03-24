# PEP-23: PASTA Production/Staging Merge	

- Author(s): Mark Servilla
- Contact: mark.servilla@mgail.com
- Status: Draft
- Type: Process
- Created: 2026-03-23
- Reviewed:
- Final:

## Introduction

This PEP proposes a process for merging the PASTA production and staging tiers into a single tier by separating uploaded data packages into "draft" (mutable) and "published" (immutable) states. Users will be able to modify draft data packages by uploading a modified version of the same revision or by removing it completely from the repository. Published data packages will have the same properties as before: an existing data package is immutable and may only be updated by providing a new revision. In addition, only published data packages will receive an DOI. The visibility of both draft and published data packages will be managed identically through the IAM service.

## Issue Statement

## Proposed Solution

### Draft Data Package

#### Properties
1. Is mutable
   1. Can be replaced with a new version of the same revision
   2. Can be deleted
2. Is a singleton: there is only one draft data package per series at any one time and must have the highest revision value of the series (i.e., at the head of the series)
3. Must have a revision value greater than its published predecessor if one exists
4. Cannot be issued a DOI (perhaps maybe a provisional DOI?)

### Published Data Package

#### Properties
1. Is immutable
2. Can have multiple "published" revisions at any one time (a series)
3. Can be issued a DOI

## Open issue(s)

1. How to display draft and published data packages?
1. How to integrate with ezEML's publication workflow?

## References

## Rejection
