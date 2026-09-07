# PEP-27: A "Human" Access Level for use in Authorization

- Author(s): Mark Servilla, Roger Dahl
- Contact: mark.servilla@gmail.com
- Status: Draft
- Type: Process
- Created: 2026-09-06
- Reviewed:
- Final:

## Introduction

As a result of robotic scrapping and DDOS attacks, EDI recently imposed access control that limited use of the PASTA API to authenticated users only—in other words, a user must be registered and signed in to access the API. This requirement served both a human verification step, greatly reducing robot access, and acquired detailed user information. However, it also broke a number of community data catalogs because their code did not comply with the sign-in requirement, resulting in "401 Not Authorized" responses from the PASTA API. Even the EDI Data Portal required modification to support API access. As a solution, EDI proposed catalogs use "zero-member" groups to reach the authenticated level when sending a request to the API (groups are equivalent to authenticated users in IAM). (Technically, the API request would include a group API access key). This approach works and allows community data catalogs and the Data Portal to access the API. However, it also broadens the definition of the "authenticated" access control level from a known trusted user to anyone passing a sophisticated (or not so sophisticated) CAPTCHA test. Simply stated, human-verified but anonymous users can now access resources that should be limited to only those who are registered and signed in. 

## Issue Statement

Today anonymous users (albeit mostly human-verified) can access resources from the EDI data repository without leaving any meaningful trace of their identity because of a loophole in the "authenticated" designation of a group within the IAM model.  Because of this regression, EDI is no longer meeting the requirement of data publishers who want data access to be available only to registered and signed-in users. This is a significant lapse within IAM's management of data access control that must be addressed quickly.  

## Proposed Solution

We propose introducing a new lower-tier access level, "human", for users who have passed the human verification check, but which have not signed in to IAM. This level will be used for groups instead of the current "authenticated" access level. We will also reset all PASTA API methods to use the "human" access level where they are currently using "authenticated." Use of the human verification check continues to meet our primary goal of blocking malicious robot calls against the API. Importantly, no changes are necessary by community data catalogs or the EDI Data Portal—the group designation to "human" will occur completely within IAM. This update will also accommodate the use of "zero-member" groups for anonymous reviewer access to selected data packages.  In this case, the group API access key will allow access to the data package under review providing the data package owner sets read access for the group.

## Open issue(s)

Use of the "human" access level would effectively limit rogue robots from penetrating the PASTA API, but it does not improve the collection of user information when high-value resources (e.g., data) are accessed. To achieve solid information collection, we recommend two modifications to data resources only: 1) remove "Public Access" and 2) add "Authenticated Access." This leaves metadata fully accessible to users with the "human" access level but forces users to sign in when attempting to access data. We would also modify IAM to display the "Authenticated Access" profile with only the "None" and "Reader" options, as is done with the "Public Access" profile, when selecting resource permissions for users and groups. This will make it easier for users to select the "authenticated" access level for data resources. For existing data packages, we could/would script modifications to resource permissions in the IAM database for all or selected data packages. For new data packages, we could/would modify the Level-0 to Level-1 transformation of metadata to require the "Authenticated Access" profile for data entities, disallowing "public" access.

## References

## Rejection
