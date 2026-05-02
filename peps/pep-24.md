# PEP-24: Ensuring user anonymity with API access keys

- Author(s): Mark Servilla, Roger Dahl
- Contact: mark.servilla@mgail.com
- Status: Draft
- Type: Process
- Created: 2026-05-01
- Reviewed:
- Final:

## Introduction

As EDI moves to require "name-based" sign in (where the user must enter a user name to authenticate) for access to all applications, the need to include anonymity as a user state for certain individuals (namely, reviewers) is critical. An alternative to "named-based" sign is already available within EDI's IAM: the group-based API access key.

## Issue Statement

Due to security and tracking concerns, EDI is rolling out "name-based" user sign in for its entire application ecosystem. The sign in process redirects the user to an OAuth identity provider or requires credentials to be entered for LDAP authentication. In either case, a user's identity, including common name and possibly other claims, are bound to the unique EDI identifier created in EDI's Identity and Access Management (IAM) system. The EDI identifier is then used internally within EDI applications for the purpose of resource access control and can be logged in both private or publicly accessible databases. If the database is publicly accessible (data download audit logs, for example), a user's identity may be easily obtained by linking EDI identities with entries displayed in resource permissions of the IAM user interface.

Although EDI does not guarantee user anonymity, it does not promote user visibility either. However, in the case of a data publication or project proposal, a user who serves the role of a reviewer and who must have access to one or more data packages should, and has the right to, expect complete anonymity so that the data package owner (or responsible parties) cannot determine personal information about the user. Moreover, EDI acknowledges that it is not sufficient to assume a users identity will be masked by hiding in plain sight among all the other users seeking data in EDI. Today, AI can easily filter out the name of any user based on patterns found in data package audit logs.
## Proposed Solution

As an alternative to "name-based" user sign in, we propose using a group assigned API access key to mask the identity of users requiring anonymity. A group assigned API access key is a standard EDI IAM API key that is linked to a user-owned group instead of an individual user. In this case, the privileges obtained using the group access key are based on the permissions assigned to the group, not the group's owner, limiting what the user of the access key can see.

The heavy lifting of this approach is already complete in IAM. The use-side of the access key, however, will require modification to any application that will accept the key in lieu of a "name-based" sign in. The PASTA "gatekeeper" already accepts the API key in support of programmatic authentication. The Data Portal does not, and will have to incorporate the access key into its authentication flow.

### Use case: data publication review

For a data producer publishing a data package in a journal, the journal will often require an anonymous reviewer to verify the integrity and veracity of the data in EDI. The following steps can be followed by the data producer to allow data package access for an anonymous reviewer (assuming that "public access" to any data objects within the data package is forbidden):

1. Create a group (using the IAM user interface).
1. Give the group access to the data object(s).
1. Create a group-based API access key for the group.
1. Provide the access key to the reviewer (or the journal on behalf of the reviewer).

## Open issue(s)

1. A proposal-based reviewer would likely require access to all data packages of a project/owner. Currently, there is not an IAM method to add a group to a set of data packages without selecting each data package individually. A "select all" option would be required in the IAM permissions management interface.

## References

## Rejection
