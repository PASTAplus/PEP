# PEP-2: Add level of abstraction between ACLs and IdPs using unique random IDs for users (PASTA ID)

- Author(s): Roger Dahl
- Contact: dahl at unm edu
- Status: Draft
- Type: Application
- Created: 2024-06-17
- Reviewed: 2024-08-28
- Final:


## Introduction

The Environmental Data Initiative (EDI) provides data repository services accessible through the Internet. Many of these services (e.g., publishing data) require an authorized identity to fulfill, an identity authenticated through one or more Identity Providers (IdPs). However, the current implementation of ACLs is tightly coupled to the unique identifier provided by the IdP. This has the potential of affecting authorization and access for the user in PASTA. This PEP proposes adding a level of abstraction between ACLs and IdPs, in the form of a randomly generated user identifier, called a user PASTA ID, to resolve this and other issues.

## Issue Statement

We support multiple IdPs for sign in. Currently, user information provided by the IdP with which the user signed in, is used directly in package entity access control lists (ACLs), and the user's PASTA token. This has the following issues:

- If the user changes their information with the IdP, such as their email address, they may lose access to their data, unless we then also update all their ACLs. Updating the ACLs probably requires manual intervention in order to verify that the 'new' user is the same as the 'old' user.

- If the IdP changes the format of the information they provide, it may cause issues similar to those caused by the user changing their own information.

- The person who assigns access to entities must know how the user signs in, and depending on that information, use their email address, ORCID, or LDAP DN in the ACLs.

## Proposed Solution

We will add a level of abstraction between ACLs and IdPs, in the form of an internal unique identifier called a PASTA ID. The PASTA ID uniquely identifies a user or group in the system. 

Instead of using information provided by the IdPs directly in the ACLs, we will use the PASTA ID in the ACLs and tokens. For users, we will store mappings to this single PASTA ID for all IdPs with which the user has signed in.

When a user signs in, the system will look up the PASTA ID for the user, based on the information provided by the IdP. If the PASTA ID is not found, a new PASTA ID will be created for the user. The system will then use the PASTA ID in the ACLs and tokens.

This resolves the issues mentioned in the Issue Statement. In addition, this has the following benefits:

- We can change or add IdPs in the future, without having to update the ACLs.

- It provides a single source of truth for information about a given user in the system.

- This change will work well with the concept of a user profile, and with group management features which we are currently planning.

- This will not prevent users from having separate identities in PASTA, as we plan on a mapping process between accounts, which will be optional.


## Open issue(s)

Implementing the proposed solution will require changes to the PASTA software. The following are some of the issues that will need to be addressed:

1. The current ACL database table contains entries based on existing IdP identifiers. We will need to add a new column to this table to store the PASTA ID and then process each unique IdP identifier to create a PASTA ID in the user profile table, if an entry does not already exist. New entries created by this process mean that the user has not yet signed in to the system (an existing entry would have been created by the user sign-in process). These entries will be incomplete until the user signs in and completes the required profile attributes not available from the ACL database table.
2. We plan to require the use of PASTA IDs in metadata-based ACLs. This will require some method (TBD) for data package authors to look up the PASTA ID for a given user. 
3. Group identifiers will be addressed in a similar manner as that for users. Groups will be assigned a PASTA ID. Issues 1 and 2 also apply to groups.


## References

