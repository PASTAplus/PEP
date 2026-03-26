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

The PASTA ecosystem, from the user's perspective, consists of two identical systems: **STAGING** and **PRODUCTION**. The first, a sandbox for testing data packages and how they are represented in PASTA and the second, the final destination for published data packages, making them available to the rest of the world. This approach works, but has a major drawback for our users: effort to test and publish a data package must be fully replicated in each system. 

Major pain points are:
1. The user must have two indentical, but separate sign-in accounts.
2. Package identifers are not synchronized across each system, leading to confusion when replicating data package from *staging* to *production*.
3. Data package uploads must be repeated in each system, requiring additional effort when a single upload may suffice.

## Proposed Solution

We propose modifying the *production* system so that data packages can exist in either a "draft" (mutable) state or a "published" (immutable) state, achieving the same goal as having *staging* and *production* environments. Critically, a *draft* data package is one that can be modified using the same data package identifier. In other words, a draft data package can be overwritten with a new version or it can be removed from the system. In contrast, a *published* data package **cannot** be modifed once "published", meaning that it receives a DOI and must (in general) be publicly accessible.

A typical user scenario is:
1. Upload a data package to PASTA.
2. PASTA sets its state to "draft".
3. User reviews data package.
4. Satisfied, the user selects the *publish* action.
5. PASTA locks the data package and assigns it a DOI, setting its state to "published".

#### "Draft" properties

1. Is mutable
   1. Can be replaced with a new version of the same revision
   2. Can be deleted
2. Is a singleton: there is only one draft data package per series at any one time and must have the highest revision value of the series (i.e., at the head of the series)
3. Must have a revision value greater than its published predecessor if one exists
4. Cannot be issued a DOI (perhaps maybe a provisional DOI?)

#### "Published" properties

1. Is immutable
2. Can have multiple "published" revisions at any one time (a series)
3. Can be issued a DOI

## Open issue(s)

1. How to display draft and published data packages?
1. How to integrate with ezEML's publication workflow?

## References

## Rejection
