# PEP-29: Double-blind Data Package Publication in PASTA

- Author(s): Mark Servilla
- Contact:
- Status: Draft
- Type: Process
- Created: 2026-10-27
- Reviewed:
- Final:

## Introduction

PASTA, in conjunction with IAM, allows for anonymous review of published data packages by creating a zero-member group API Access Key and assigning it to the reviewers. PASTA does not, however, support double-blind reviews, where both the data authors and the reviewers are anonymous. This PEP proposes a workaround to enable double-blind reviews in PASTA.

## Issue Statement

A double-blind review of a published data package in PASTA requires that both the list of authors and reviewers remain anonymous throughout the review process. IAM's use of [zero-member group API Access Keys](https://edirepository.org/user-guides/iam#working-with-groups) allows reviewers to remain anonymous through the group access key&mdash;common names or personal identities are never employed in this process. (The data package, including the data entities, **SHOULD BE** restricted to only the data package owner and the group during such reviews.)

Keeping the data authors anonymous during the review process, however, is more challenging: Because of PASTA's data package immutability principle requiring that published data packages cannot be altered, the data package&mdash;the one under review&mdash;must be the final version. To be metadata complete, this data package will contain author names and very likely those names of related personnel or other identifiable information, which cannot be discerned in a double-blind review. 

## Proposed Solution

The proposed solution is quite simple:

1. Have the data author create both a metadata complete version of the EML and a redacted version. The complete version *SHOULD NOT* include any `<access>` element, rendering the full data package inaccessible to everyone except for the submitter immediately after publication.
2. Publish the data package using the complete version of the EML. Because of the restricted access, PASTA will not register a DOI with DataCite, nor will it synchronize it with DataONE. PASTA will, however, record all other necessary information, making this a fully functional data package, albeit with access restricted to only the submitter.
3. Swap the redacted version of the EML with the complete version using the exact same name: `Level-1-EML.xml`. This step must be performed by a system administrator. **Be sure the original complete version of the EML is saved to another name (`Level-1-EML.xml.original`, for example) before swapping.**
4. Now that the redacted version of the EML is in place, add permissions for the group to enable read access to the data package, allowing the reviewers access to the data package. At this point, both the data authors (vis-à-vis the redacted EML) and the reviewers are anonymous.
5. If a DOI must be assigned to the data package for the review to proceed, a provisional DOI can be generated and provided to the data authors. (This DOI, however, will not be registered with DataCite until the data package metadata is made publicly accessible.)
6. Once the review is complete, replace the redacted version of the EML with the original complete version and set the data package permissions accordingly&mdash;the metadata and report should be publicly accessible, along with any data not embargoed for other reasons. (Public metadata is required for DataCite DOI registration, while public metadata and data are required for DataONE synchronization.) Once publicly accessible, PASTA will register the data package DOI with DataCite, and if fully accessible, the data package will be synchronized with DataONE.

## Open issue(s)

## References

## Rejection
