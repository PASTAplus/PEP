# PEP-15: Authentication with API Keys

- Author(s): Mark Servilla
- Contact: mark.servilla@gmail.com
- Status: Draft
- Type: Application
- Created: 2025-01-25
- Reviewed:
- Final:


## Introduction

The Environmental Data Initiative (EDI) will soon be requiring users to authenticate before being permitted to download data, including downloading a zip archive of the data package, which contains all the package data (see [PEP-14](./pep-14.md)). User authentication is easily managed in EDI's many application user interfaces (UI), but not 

## Issue Statement

This change will occur at the point when authorization of a user's request is processed to determine if the request has permission to execute the `readDataEntity` and `downloadZipArchive` REST API methods of the Data Package Manager web service. Authorization is performed by analyzing the user's PASTA authentication token, which is passed to the Data Package Manager by the Gatekeeper reverse proxy service.


## Proposed Solution

...

## Open issue(s)

...

## References

...

## Rejection

...
