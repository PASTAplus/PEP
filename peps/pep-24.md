# PEP-24: Ensuring user anonymity with API access keys

- Author(s): Mark Servilla, Roger Dahl
- Contact: mark.servilla@mgail.com
- Status: Draft
- Type: Process
- Created: 2026-05-01
- Reviewed:
- Final:

## Introduction

As EDI moves to require "named" sign in authentication for access to all applications, the need to include anonymity as a user state for certain individuals (namely, reviewers) is critical. An alternative to "named" sign in must be found. This alternative is already present within EDI's IAM as the group-based API access key, which works today on PASTA's REST API interface. Minor modification to the Data Portal will also allow the API access key to be used in lieu of a "named user" sign in.

## Issue Statement

Due to security and tracking concerns, EDI is slowly enforcing user "sign in" for its entire application ecosystem.

 It is not sufficient to assume a users identity will be masked by hidding in plain sight amongst all the other users seeking data in EDI. Today, AI can easily filter out the name of any user based on patterns found in data package audit logs.
## Proposed Solution

## Open issue(s)

## References

## Rejection
