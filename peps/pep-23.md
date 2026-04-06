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

Major "user" pain points are:
1. The user must maintain two indentical, but separate sign-in accounts.
2. Data package identifers are not synchronized across each system, leading to confusion when replicating data package from *staging* to *production*.
3. Data package uploads must be repeated in each system, requiring additional effort when a single upload may suffice.

Major "EDI" pain points are:
1. Infrastructure support for the *staging* tier: PASTA staging, Portal staging, and IAM staging. 
2. ezEML workflow coordination.

## Proposed Solution

We propose modifying the *production* tier so that data packages can exist in either a "draft" (mutable) state or a "published" (immutable) state, achieving the same goal as having both the *staging* and *production* tiers.

Critically, a *draft* data package is one that can be modified using the same data package identifier. In other words, a draft data package can be overwritten with a new version or it can be removed from the system completely without consequence. In contrast, a *published* data package **cannot** be modifed once in the "published" state, allowing it to receive a DOI. A published data package must also be publicly accessible (with some exceptions).

A typical user scenario is:
1. Upload a data package to PASTA.
2. PASTA sets its state to "draft".
3. User reviews data package.
4. Satisfied, the user selects the *publish* action.
5. PASTA locks the data package making it immutable and assigns it a DOI, setting its state to "published".

#### "Draft" properties

1. A draft data package is mutable:
   1. Can be replaced with a new version using the same data package identifier.
   2. Can be deleted.
2. There can be only one draft data package per series at any one time; this draft data package must have the highest revision value of the series (i.e., at the head of the series).
3. A draft data package must have a revision value greater than any published predecessor (if one exists).
4. A draft data package is not issued a DOI.

#### "Published" properties

1. A published data package is immutable.
2. Multiple published data packages in the same series may exist.
3. A published data package is issued a DOI.

### Rollout Plan

1. **Incremental**

An incremetnal release process would bring all changes into, but not beyond, the existing staging system. This would allow users to experience the "merged" system, while continuing to perform typical data package sandboxing. No changes will be added to the production environment until a complete merge is operational in the staging system.

2. **Binary**

A binary release process would include all changes into all systems (development, staging, and production) as they are tested and deployed, but surpress their actions until a decision is made to switch from "production + staging" to only "production" (the merged version).

## Open issue(s)

1. How to display draft and published data packages?
1. How to integrate with ezEML's publication workflow?
1. Should a draft data package have a provisional DOI?
1. What API changes will be required to support the merge?
   1. For example, "publish"

## References

## Rejection
