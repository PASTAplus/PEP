# PEP-14: Requiring Authentication for Data Access

- Author(s): Mark Servilla
- Contact: mark.servilla@gmail.com
- Status: Draft
- Type: Policy
- Created: 2025-01-18
- Reviewed:
- Final:


## Introduction

The Environmental Data Initiative proposes changing rights for accessing data from "public" *READ* to "authenticated" *READ*, thereby eliminating unfettered data exfiltration by non-identified agents.  

## Issue Statement

To date, accessing and downloading data through the Environmental Data Initiative data repository requires no proof of identity. As such, data exfiltration, whether legitimate or malicious, can occur without the knowledge of the downloading agent. This leads to two issues. From a business perspective, EDI knows nothing about the market segment that is downloading and using the data curated by EDI. This information is critical to ensure EDI services correctly address consumer needs. In addition, many data contributors have requested that EDI capture and provide more details about who is downloading and using their data. (Today, this can only be done on a case-by-case basis through resource-based access control rules, which would be the responsibility of the data contributor.) More concerning, however, is the exfiltration of data *en masse* by web scrapers or robots.  These non-human applications download data to benefit their masters' needs (e.g., AI model building, unsolicited replication, ...) and seldom promote new or ongoing environmental research.

## Proposed Solution

We propose to use existing technology already built into the PASTA data repository software's identity and access management model by modifying the access control rules for the `readDataEntity` and `downloadDataPackageArchive` service methods to remove the **READ** permission for the "public" (i.e., anonymous) user and add the **READ** permission for the "authenticated" user group, requiring users to authenticate using either an EDI LDAP user account or an OAuth relying party account (e.g., GitHub, Google, Microsoft, or Orcid) to download data from either the EDI Data Portal or the REST API.

For non-authenticated users, the EDI Data Portal will display all elements of the data package landing page, with the exception of providing links to download the data package archive or individual data files. If the user authenticates, they will be returned to the landing page previously viewed. To download the data package archive or individual data files from the REST API, users will have to provide either LDAP credentials in the **authorization header** of the HTTP request or a valid PASTA authentication token in the **cookie header** of the same request. Both options work today without any additional software modification.

## Open issue(s)

### Limitations on autonomous data download requests

Existing workflows that rely on non-authenticated access to data downloads will be required to add either EDI credentials or a PASTA authentication token to the HTTP request. This poses the following issues:
1. EDI credentials are not optimal for temporary users, such as reviewers who need access to data only during manuscript reviews. Creating temporary credentials and adding these to the HTTP request for a reviewer would be time-consuming, error-prone, and just arduous. 
2. PASTA authentication tokens are short-lived and would require continual refresh by the client software, especially if the applications are autonomous such workflows that are executed on a timed or event basis. 

For these reasons, EDI will explore the use of API keys that can be assigned to an autonomous service in lieu of a human user. See [PEP-15](./pep-15.md) for a detailed discussion about the use API Keys for the EDI IAM model.

## References

...

## Rejection

...
