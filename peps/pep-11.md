# PEP-11: AWS S3 Plan for EDI Data Resources

- Author(s): Mark Servilla
- Contact: mark.servilla@gmail.com
- Status: Draft
- Type: Process
- Created: 2024-12-08
- Reviewed:
- Final:


## Introduction

The PASTA software stack currently uses block storage for all data resources published to the Environmental Data Initiative (EDI) data repository. EDI is migrating PASTA services (virtual machines, system storage, and data storage) to Amazon Web Services (AWS). As part of this move, EDI plans to utilize AWS's Scalable Storage System (S3) for all published data resources. The following proposal outlines the organization and storage classification of data resources using objects within S3 buckets.

## Issue Statement

The EDI data repository, which uses the PASTA software stack for repository services, has operated on premise at the University of New Mexico since January 2013. Published data in the repository has always utilized local data storage arrays mounted to servers as block storage devices. Data resources of the published data in the EDI data repository are written to directories of the data package name (e.g., `knb-lter-nin.3.1`) that conforms to the "scope.identifier.revision" convention. Data resources include:

1. `Level-0-EML.xml`: EML metadata document submitted by author
2. `Level-1-EML.xml`: EML metadata document modified by EDI for publication
3. `Level-1-DC.xml`: Dublin Core metadata based on `Level-1-EML.xml`
4. `quality_report.xml`: EML congruency checker quality report
5. `673770d3d0507bc0c2cac3eaccd71e70`: one or more data objects with hashed names

For reasons beyond the scope of this proposal, EDI has decided to migrate it services, including all data resources, to AWS. Data resources will exist in AWS S3 as objects within buckets. This proposal presents an approach for organizing data resources in S3, including strategies for initial loading of data resources and how the PASTA software stack will interact with S3.

## Proposed Solution

...

## Open issue(s)

...

## References

...

## Rejection

...
