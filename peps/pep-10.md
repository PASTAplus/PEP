# PEP-10: Improve Data Package License Machine Readability with SPDX

- Author(s): Colin Smith
- Contact: colin.smith@wisc.edu
- Status: Draft
- Type: Application/Policy
- Created: 2024-12-05
- Reviewed:
- Final:


## Introduction

Data package licensing is crucial for clearly communicating data usage terms to consumers. Currently, EDI assists data authors by presenting a couple license options aligned with open data principles, along with a third option for custom content. This information is stored in EML's [intellectualRights](https://eml.ecoinformatics.org/schema/eml-resource_xsd#ResourceGroup_intellectualRights) element, which is encouraged by EDI policy.

However, the intellectualRights element accepts loosely formed [TextType](https://eml.ecoinformatics.org/schema/eml-text_xsd.html#TextType) metadata. While human-readable, this format is largely unintelligible for machines and hinders automated license interpretation.

_Note, this proposal was first introduced as a GitHub issue in EDI's EML Best Practices repository. See EDIorg/data-package-best-practices#86_

## Issue Statement

This PEP improves the machine readability of data package licenses by proposing the use of SPDX license identifiers in EML metadata. SPDX identifiers are machine-resolvable URIs that represent standardized license terms. By adopting SPDX identifiers, EDI can enhance license interoperability and support automated license interpretation across systems.

## Proposed Solution

To enhance current practices, and align with [Science-On-Schema.org](https://github.com/ESIPFed/science-on-schema.org/blob/master/guides/Dataset.md#license) practices for license interoperability, consider encouraging the use of EML's [licensed](https://eml.ecoinformatics.org/schema/eml-resource_xsd#ResourceGroup_licensed) element. This element accepts a URL to a machine-resolvable, linked data compliant license. EDI can offer SPDX license identifiers as linked data URIs alongside the current practices using the intellectualRights element. Eventually, phasing out the free-text intellectualRights element might be considered (see note below).

The current set of EDI-recommended licenses and their corresponding Creative Commons and SPDX identifiers are:

__Creative Commons Zero v1.0 Universal__
- https://creativecommons.org/publicdomain/zero/1.0/
- https://spdx.org/licenses/CC0-1.0

__Attribution 4.0 International__
- https://creativecommons.org/licenses/by/4.0/
- https://spdx.org/licenses/CC-BY-4.0

An example of how this looks in EML:

```xml
<licensed>
  <licenseName>Creative Commons Zero v1.0 Universal</licenseName>
  <url>https://spdx.org/licenses/CC0-1.0</url>
  <identifier>CC0-1.0</identifier>
</licensed>
```

Choosing between Creative Commons and SPDX may have future implications for supporting other licenses. SPDX encompasses all Creative Commons licenses and offers greater flexibility for accommodating additional licenses in the future.

_Note: Phasing out the free-text intellectualRights element would eliminate potential contradictions with the licensed element and limit support for arbitrary, non-standardized data use terms. However, it may restrict the ability to express nuanced usage rights not yet formalized by the broader community._

## Open issue(s)

Some affected systems include:
- EDI systems ([PASTA](https://github.com/PASTAplus/PASTA), [DataPortal](https://github.com/PASTAplus/DataPortal))
- Metadata editors ([ezEML](https://github.com/PASTAplus/ezEML), [metapype-eml](https://github.com/PASTAplus/metapype-eml), [EMLassemblyline](https://github.com/EDIorg/EMLassemblyline))
- [Data Package Best Practices](https://github.com/EDIorg/data-package-best-practices)