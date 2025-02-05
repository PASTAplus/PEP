# PEP-15: Authentication with API Keys

- Author(s): Mark Servilla
- Contact: mark.servilla@gmail.com
- Status: Draft
- Type: Application
- Created: 2025-01-25
- Reviewed:
- Final:


## Introduction

The Environmental Data Initiative (EDI) will soon require users to authenticate before being permitted to download data (see [PEP-14](./pep-14.md)), including zip archives of data packages (which contain all the package data). 

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
