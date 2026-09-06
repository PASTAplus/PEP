# PEP-27: A "Human" Access Level for use in Authorization

- Author(s): Mark Servilla, Roger Dahl
- Contact: mark.servilla@gmail.com
- Status: Draft
- Type: Process
- Created: 2026-09-06
- Reviewed:
- Final:

## Introduction

As a result of robotic scrapping and DDOS attacks, EDI recently imposed a requirement that limits access to the PASTA API by authenticated users only. This requirement serves both a human verification step, reducing robot access, and provides detailed user information to better understand our target demographic. However, this requirement also forces users to sign in and create an account in EDI's Identity and Access Manager (IAM) when accessing repository content, including through the Data Portal and community-supported data catalogs. To facilitate a community request allowing human-verified anonymous access to low-value content, like metadata, "zero-member" groups, along with API access keys, were proposed as a novel solution for passing the authentication barrier since groups are naturally an authenticated entity in the IAM system. Although this approach works to enable Data Portal and catalog access, it also broadens the definition of "authenticated" from a known user with trusted credentials to anyone passing a sophisticated (or not so sophisticated) CAPTCHA test. A better approach is to separate the user-base into human-verified but anonymous users and those who have a registered user account in EDI, allowing EDI to restrict high-value content to only authenticated users. 

## Issue Statement

Today anonymous users, albeit mostly human-verified, can access high-value content from the EDI data repository without leaving any meaningful trace of their identity because of a loophole in the "authenticated" designation of a group within the IAM model. Although robots are effectively blocked, EDI's intention to secure more detailed user information when accessing high-value data is thwarted. In addition, EDI is no longer meeting the requirement of data publishers who want data access to be available only to signed-in users. This is a significant issue within IAM's management of data access control that must be addressed quickly.  

## Proposed Solution

We propose introducing a new system category, “human,” to replace “authenticated” across all user groups (including "zero-member" groups). A “human” verified user validates that a requester is not a robot, without requiring registration in EDI’s IAM system. This allows EDI to restrict high-value data to registered, authenticated users while still providing human-verified anonymous access to low-value resources, such as metadata. To enable this, EDI will adjust API access requirements so that all endpoints not serving high-value data require only “human” verification (as designated by a "zero-member" group), while direct data access will require an “authenticated” user. (Services requiring the “vetted” access, such as data publication or updates, will remain unchanged.)

## Open issue(s)

This approach **will** affect the Data Portal and community catalogs that rely on "zero-member" groups meeting the "authenticated" level for data access. To address this issue, the Data Portal will redirect users to the login page if they only meet the “human” access level, allowing them to sign in to IAM and secure an "authenticated" level of access.

For community catalogs based on ezCatalog, which redirects users to the Data Portal, the Data Portal will handle redirection to the login page. The catalog requires nothing else and can operate as is. If, however, the catalog exposes API-based data links to the user, IAM will block access to the data download. In this case, we propose the following:

1. Allow the "human" access level permission to use the data download API for a 3-month grace period. Regardless, the Data Portal will immediately redirect users to the login page if they only meet the “human”  level for data access.
2. Have community catalogs explicitly provide read-access to data packages they own for their "zero-member" groups.

However, providing explicit data package read-access to a "zero-member" group means that users *will be able* to access both low and high-value resources without signing-in to IAM. Detailed user information **will not** be available for these access events. Lack of data access user information is the responsibility of the catalog that enables read-access for the data they own and control.  

## References

## Rejection
