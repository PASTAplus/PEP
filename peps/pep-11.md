# PEP-11: AWS S3 Plan for EDI Data Resources

- Author(s): Mark Servilla
- Contact: mark.servilla@gmail.com
- Status: Draft
- Type: Process
- Created: 2024-12-08
- Reviewed:
- Final:


## Introduction

The PASTA software stack currently uses block storage for all data resources published to the Environmental Data Initiative (EDI) data repository. EDI is migrating PASTA services (virtual machines, system storage, and data storage) to Amazon Web Services (AWS). As part of this move, EDI plans to utilize AWS's Simple Storage Service (S3) for all published data resources. The following proposal outlines the organization and storage classification of data resources using objects within S3 buckets.

## Issue Statement

The EDI data repository, which uses the PASTA software stack for repository services, has operated on premise at the University of New Mexico since January 2013. Published data in the repository has always utilized local data storage arrays mounted to servers as block storage devices. Data resources of the published data in the EDI data repository are written to directories of the data package name (e.g., `knb-lter-nin.3.1`) that conforms to the "scope.identifier.revision" convention. Data resources include:

1. `Level-0-EML.xml`: EML metadata document submitted by author
2. `Level-1-EML.xml`: EML metadata document modified by EDI for publication
3. `Level-1-DC.xml`: Dublin Core metadata based on `Level-1-EML.xml`
4. `quality_report.xml`: EML congruency checker quality report
5. `673770d3d0507bc0c2cac3eaccd71e70`: one or more data objects with hashed names

For reasons beyond the scope of this proposal, EDI has decided to migrate it services, including all data resources, to AWS. Data resources will exist in AWS S3 as objects within buckets. This proposal presents an approach for organizing data resources in S3, including strategies for initial loading of data resources and how the PASTA software stack will interact with S3.

## Proposed Solution

We propose to replace the hierarchical structure of the current EDI data resource block storage organization with a similar structure within multiple AWS S3 buckets. Buckets are logical containers for data objects residing in AWS S3 and are analogous to a mounted file system in modern desktop or server computers. In general, buckets have few constraints (see [working with buckets](https://docs.aws.amazon.com/AmazonS3/latest/userguide/creating-buckets-s3.html)) and support the concept of folders (i.e., directories), which can be nested within other folders, providing the same hierarchical layering available in a block storage device. We plan to organize buckets around EDI services, naming them with some element of the service being represented. Because buckets names must be unique across all of AWS (see [bucket name rules](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html) for more information), we will use a combination of the EDI service name prefixed to a UUID, creating unique EDI bucket names. For example, EDI maintains three separate physical tiers of the PASTA software, one each for development, staging, and production uses, respectively. Corresponding buckets names would be:

1. `edi-pasta-development-0c8c25f7-82f9-4350-99d8-9cd4579c8b92`
2. `edi-pasta-staging-28a0a25b-33d0-4560-afb2-abd6745aae67`
3. `edi-pasta-production-bb417e09-4b3f-46be-8138-f5f83da77493`

Similarly, we would create a bucket named `edi-ezeml-307e0697-c244-4730-904b-4bacd92fc2eb` for "ezEML" storage needs.

## Open issue(s)

...

## References

...

## Rejection

...
