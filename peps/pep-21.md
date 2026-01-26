# PEP-21: Declaring Resource Relationships via Sidecar Metadata Database

- Author(s): Colin Smith
- Contact: colin.smith@wisc.edu
- Status: Draft
- Type: Application | Process | Policy
- Created: 2026-01-05
- Reviewed:
- Final:

## Introduction

This PEP addresses the technical challenge of declaring relationships between EML metadata records and related resources (e.g., other data packages or publications) without triggering an infinite DOI versioning loop. It proposes a transition from hardcoded version-specific identifiers within EML to a system of semantic annotations ingested into a "sidecar" database to power human-readable discovery, attribution, and machine-harvesting.

## Issue Statement

The current reliance on version-specific Persistent Identifiers (PIDs) for cross-referencing resources in EML metadata creates a **circular dependency loop**. When Data Package A references a specific version of Resource B, any update to Resource B (to point back to Data Package A) generates a new DOI for Resource B. This, in turn, necessitates an update to Data Package A’s metadata to reflect the new DOI, triggering a new DOI for Data Package A.

**The primary challenges are:**

* **Human Visibility Gap:** Currently, it is difficult for a researcher visiting a data package landing page to quickly see and navigate to related resources because those relationships are often buried in immutable, version-locked XML.
* **Version Explosion:** Metadata records are frequently updated solely to repair links, leading to unnecessary version inflation and "administrative" versions that do not contain scientific changes.
* **Maintenance Burden:** Researchers using **ezEML** or programmatic scripts, to generate EML, must manually track and update DOIs across multiple platforms to maintain connectivity.

**Benefits of resolution:**

* **Human Discovery:** Enables the landing page to dynamically display a "Related Resources" section, allowing researchers to see the "web" of connectivity between data and other assets.
* **Stability:** EML records remain valid across resource versions by referencing series-level identifiers.
* **Attribution:** Facilitates the automatic pushing of relationship data to DataCite for citation tracking and credit.
* **Searchability:** Provides a mechanism to inject relationships into Schema.org JSON-LD for global search aggregators.


## Proposed Solution

### Technical Description

The proposal moves toward a **Hybrid Metadata Architecture** where the sidecar database acts as the primary source of truth for declaring relationships. Users are expected to declare this information directly in the sidecar database via its user interface or API. This is the short-term priority, as it simplifies the workflow and ensures consistency.

In the long term, we may expose the ability for users to declare relationships directly in EML metadata records. This information could then be harvested into the sidecar database during the ingestion process. However, this capability is not part of the current implementation plan.


**Use of Data Package Series URLs:** For internal references, users are directed to use the "Data Package Series URL" (Concept URL) rather than a versioned DOI.
   * *Example:* `https://portal.edirepository.org/nis/mapbrowse?scope=knb-lter-mcm&identifier=501`

A long-term goal could be a transition to using Data Package Series DOIs, which would provide a more robust and persistent solution for referencing the newest version of a data package series.

**Multi-Channel Exposure:** The sidecar database will dynamically feed:
   * **Landing Page UI:** A human-readable section showing all related resources.
   * **Schema.org JSON-LD:** Metadata embedded in the HTML for harvesting by Google Dataset Search.
   * **DataCite Metadata:** Updating `RelatedIdentifier` fields via API to ensure proper citation counts.
   * **EML Records:** Information from the sidecar database will be injected into EML records downloaded by users. This preserves the relationships declared in the sidecar database and ensures this information is available for workflows using the downloaded EML.

### Business Case

This approach delivers high-value discovery features to human consumers while lowering the cognitive load for **EML** creators. It provides a "set it and forget it" linking mechanism that supports the research lifecycle. By decoupling the relationship from the EML bits, we ensure that EDI provides high-fidelity attribution data to the global PID graph without the overhead of constant re-versioning.

### Use Case Example

1. Users upload a data package to the EDI data repository.
2. They declare related resources to that data package through the sidecar database UX or API.
3. This information becomes accessible to viewers of the data package landing page.
4. The information is embedded in the EML when a user downloads the EML, where semantic annotations declare all the related resources.

### Example EML Snippet

```xml
<dataset>
  <title>Sample LTER Dataset</title>
  <!-- Placeholder for additional metadata -->
  <annotation>
    <propertyURI label="isRelatedTo">http://purl.org/dc/terms/relation</propertyURI>
    <valueURI label="Related Dataset Series">https://portal.edirepository.org/nis/mapbrowse?scope=knb-lter-mcm&amp;identifier=501</valueURI>
  </annotation>
</dataset>
```

### Potential Negative Impacts

* **Database Load:** Minor increase in relational database size and query complexity to render landing pages.
* **Ingestion Latency:** Negligible overhead during the EML parsing phase.

### Effort Estimation

* **Medium:** Requires updates to the PASTA ingestion service, database schema modifications, landing page renderer updates, and ezEML interface support.

### Alternatives Considered

* **Manual DOI Management:** Rejected due to the circular dependency loop.
* **Non-Semantic Text References:** Rejected as it prevents machine-readability and automated citation tracking.


## Open issue(s)

* **Standard Set of URIs:** The ETSC must agree on a **standard set of URIs** for the annotation properties (predicates) to handle various relationship types (e.g., `isRelatedTo`, `isSupplementTo`, `cites`).
* **Mapping Failures:** A process must be defined for when a Series URL fails to resolve or is entered incorrectly. Mitigation includes a "Report Discrepancy" feature on the landing page allowing for manual backend correction in the sidecar database by the EDI team.
* **DataCite API Constraints:** Confirmation that DataCite administrative updates can be performed via API without triggering version increments for all supported member nodes.

## References

* [Ecological Metadata Language (EML) 2.2.0 Specification](https://doi.org/10.5063/F11834T2).
* [DataCite Metadata Schema Documentation](https://support.datacite.org/docs/datacite-metadata-schema).
* [Structured Data for Dataset Provenance (Google, Schema.org)](https://developers.google.com/search/docs/appearance/structured-data/dataset#source-provenance).
