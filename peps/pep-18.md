# PEP-18: Automatic QUDT Units Annotation in ezEML

- Author(s): Jon Ide
- Contact: jride@wisc.edu
- Status: Draft
- Type: Application
- Created: 2025-06-11
- Reviewed:
- Final:

## Introduction

The EML standard provides for defining units (standard or custom) for numerical values. 

The EDI Units Working Group has done extensive, excellent work on mapping commonly used units to the QUDT ontology. These mappings can be used to annotate attributes, thereby enhancing the units information.

This document describes how ezEML may be modified to employ the Units WG's QUDT mappings, where available, to automatically insert annotations into EML metadata.

## Issue Statement

Units annotations are valuable additions to the metadata describing data entities. Constructing such annotations manually, however, is a prohibitively onerous and error-prone task. By leveraging the EDI Units Working Group's work, ezEML can automically add units annotations in the 90-95% of cases where a mapping exists.

## Proposed Solution

EDI will utilize the Units WG's data files, hosted on GitHub. EDI will substitue Python versions of the R services authored by John Porter and host these services on EDI servers. The GitHub data files will also be copied to EDI servers periodically so that EDI is not dependent on internet access either to John Porter's servers or GitHub's. The EDI services will be exposed as REST endpoints so they can be used by other EDI applications in addition to ezEML.

One such service will accept an EML file and return an annotated EML file and, optionally, the corresponding MetaPype JSON file. Another such service will return the annotation for a single unit instance.

When a package is fetched or imported into ezEML, ezEML will add annotations to attributes (unless the user opts out via the Settings page). These will be added within the attribute blocks rather than in a separate annotations section. Users will have the option of replacing existing annotations or not if some QUDT units annotations already exist (again, via the Settings page).

In addition, attribute IDs will be added to all attributes, and the EML version will be updated to EML.2.2.

The Check Metadata page will have a button to Review QUDT Units. This will bring up a page displaying a table with columns:

- Data table name
- Column name
- Unit as entered
- QUDT code
- Option to reject the QUDT code

The user can reject a code that seems incorrect but will not have the ability to enter a corrected annotation. 

The QUDT code may also be displayed on the Column Properties page for each column. TBD.

In addition to creating annotations when fetching/importing, ezEML will create annotations when standard or custom units are entered via the ezEML UI. It will be useful, accordingly, to have a REST endpoint that handles an individual unit, in addition to the one that handles an entire EML file.

ezEML will log the following cases:

- a QUDT mapping was not found
- a QUDT mapping was rejected by the user
- a QUDT annotation already exists but is different from our mapping

## Open issue(s)

What should attribute IDs look like? GUIDs have much to recommend them but are a bit ugly.

Should we enable the user to enter a different QUDT code?

## References

## Rejection
