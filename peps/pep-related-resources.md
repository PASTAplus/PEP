# PEP-N: Decoupling Resource Relationships via Semantic Annotations and Sidecar Metadata

- Author(s): Colin Smith
- Contact: colin.smith@wisc.edu
- Status: Draft
- Type: Application | Process | Policy
- Created: 2026-01-05
- Reviewed:
- Final:

## Introduction

This PEP addresses the technical challenge of declaring relationships between EML metadata records and related resources (e.g., other datasets or publications) without triggering an infinite DOI versioning loop. It proposes a transition from hardcoded version-specific identifiers within EML to a system of semantic annotations ingested into a "sidecar" database to power human-readable discovery, attribution, and machine-harvesting.

## Issue Statement

The current reliance on version-specific Persistent Identifiers (PIDs) for cross-referencing resources in EML metadata creates a **circular dependency loop**. When Dataset A references a specific version of Resource B, any update to Resource B (to point back to Dataset A) generates a new DOI for Resource B. This, in turn, necessitates an update to Dataset A’s metadata to reflect the new DOI, triggering a new DOI for Dataset A.

**The primary challenges are:**

* **Human Visibility Gap:** Currently, it is difficult for a researcher visiting a dataset landing page to quickly see and navigate to related resources because those relationships are often buried in immutable, version-locked XML.
* **Version Explosion:** Metadata records are frequently updated solely to repair links, leading to unnecessary version inflation and "administrative" versions that do not contain scientific changes.
* **Maintenance Burden:** Researchers using **ezEML** or programmatic scripts, to generate EML, must manually track and update DOIs across multiple platforms to maintain connectivity.

**Benefits of resolution:**

* **Human Discovery:** Enables the landing page to dynamically display a "Related Resources" section, allowing researchers to see the "web" of connectivity between data and other assets.
* **Stability:** EML records remain valid across resource versions by referencing series-level identifiers.
* **Attribution:** Facilitates the automatic pushing of relationship data to DataCite for citation tracking and credit.
* **Searchability:** Provides a mechanism to inject relationships into Schema.org JSON-LD for global search aggregators.


## Proposed Solution

### Technical Description

The proposal moves toward a **Hybrid Metadata Architecture** where the EML record acts as the "instruction set" for a dynamic relationship graph stored in a sidecar database.

1. **Annotation-Based Declaration:** Users declare relationships in EML using the `<annotation>` element. This could be supported both in the **ezEML** UI and programmatic EML generation (R/Python).
2. **Use of Dataset Series URLs:** For internal references, users are directed to use the "Dataset Series URL" (Concept URL) rather than a versioned DOI.
   * *Example:* `https://portal.edirepository.org/nis/mapbrowse?scope=knb-lter-mcm&identifier=501`

3. **Smart Ingestion Engine:** Upon upload, PASTA will "pluck" these annotations and store them in a relational sidecar database.
4. **Normalization Logic:** The system will resolve the Dataset Series URL to the most recent versioned DOI at the time of publication for administrative metadata tasks like DataCite registration.
5. **Multi-Channel Exposure:** The sidecar database will dynamically feed:
   * **Landing Page UI:** A human-readable section showing all related resources.
   * **Schema.org JSON-LD:** Metadata embedded in the HTML for harvesting by Google Dataset Search.
   * **DataCite Metadata:** Updating `RelatedIdentifier` fields via API to ensure proper citation counts.



### Business Case

This approach delivers high-value discovery features to human consumers while lowering the cognitive load for **EML** creators. It provides a "set it and forget it" linking mechanism that supports the research lifecycle. By decoupling the relationship from the EML bits, we ensure that EDI provides high-fidelity attribution data to the global PID graph without the overhead of constant re-versioning.

### Use Case Example

A researcher updates a long-term monitoring dataset. They want to link it to a previous related study. Instead of finding the specific DOI for the current version of that study, they enter the Dataset Series URL into their preferred EML editor. PASTA ingests this EML, identifies the relationship, and automatically displays a link to the latest version of the study on the dataset's landing page.

### Example Implementation (EML Snippet)

```xml
<dataset>
  <title>Sample LTER Dataset</title>
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


