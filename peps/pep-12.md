# PEP-12: Shadow Metadata Overview



* Author(s): Colin Smith
* Contact: colin.smith@wisc.edu
* Status: Draft
* Type: Application, Process, Policy
* Created: 2024-12-18
* Reviewed:
* Final:


## Introduction

This PEP outlines a proposal for implementing "Shadow Metadata" within the EDI repository. This feature aims to enhance the discoverability, usability, and long-term value of existing data packages while respecting the principle of immutability for published EML metadata.


## Issue Statement

The EML metadata published with data packages in the EDI data repository represents a valuable resource. However, the immutable nature of this metadata can limit our ability to adapt to evolving needs and technologies. A few scenarios highlight the need for metadata augmentation:



* **Adding Value to Existing Records:** For example, enhancing metadata with semantic annotations can significantly improve both human and machine understanding. Adding controlled vocabulary terms (e.g., from a thesaurus or ontology) to describe ecological concepts, geographic locations, or instrument types can improve search accuracy and interoperability.
* **Updating Outdated Information:** In cases where the original data author is unavailable or unwilling to make corrections, there may be errors or outdated information in the metadata. Having a mechanism to address these issues without involving the original author or altering the original record can be important.
* **Modernizing Legacy Metadata:** As technology and best practices evolve, older metadata records may become less effective. Being able to update these records to current standards, ensures their continued usability.

Addressing these issues will improve the discoverability and reusability of data, better enabling EDI to meet the research community's needs. It also allows EDI to remain agile and adapt to new technologies. However, any solution must respect the principles of immutability that underpin the EDI repository.


## Proposed Solution

The proposed solution involves creating a layer of mutable "Shadow Metadata" associated with each data package. This separate metadata layer can be updated without affecting the original published EML. Note, Shadow Metadata refers to the content, which may be stored in a relational database, or found in an augmented EML document. In the latter the augmented document would be referred to as "Shadow EML".

Shadow metadata could be generated through different methods:



* **Algorithmic Generation:** Automated processes could extract information from the data package (e.g., file formats, data types, keywords) to generate shadow metadata. More advanced algorithms could be used for tasks like topic modeling or entity recognition.
* **Curatorial Input:** Data curators could manually add or refine shadow metadata to provide expert context, correct errors, or add semantic annotations.
* **User Contributions (Potential Future Enhancement):** In the future, a mechanism could be explored to allow community contributions to shadow metadata, with appropriate moderation and quality control.

Shadow metadata would be presented to users alongside the original data package, with clear visual cues to distinguish between the two. For example, a separate tab or section could display the shadow metadata, clearly labeled as such, and provide information about its source and creation date.


### Benefits



* **Enhanced Discoverability:** Semantic annotations and updated metadata improve search accuracy and retrieval.
* **Improved Usability:** Additional context and information make data packages easier to understand and use.
* **Preservation of Original Metadata:** The immutability of the original EML is maintained, ensuring data integrity and provenance.
* **Agility and Adaptability:** EDI can more easily integrate with new technologies and evolving community needs.


### Potential Drawbacks and Challenges



* **Increased Complexity:** Managing two layers of metadata adds complexity to the system.
* **Potential for Conflicting Information:** Clear policies and procedures are needed to manage potential conflicts between original and shadow metadata.
* **Data Quality Control:** Processes for ensuring the quality and accuracy of shadow metadata are crucial.


## Open Issue(s)

There are several issues regarding Shadow Metadata that will need to be resolved for effective implementation. Each item in the following list will be addressed in separate, more narrowly scoped PEPs:



* **Policies and Practices:** Formulating policies and practices for generating shadow metadata to ensure effective and responsible implementation.
* **Generation:** Generating shadow metadata content using methods that minimize the risk of inaccurate and misleading information.
* **Storage:** Storing shadow metadata and relevant ancillary information in a flexible and scalable manner.
* **Schema:** Creating or adopting a schema for the shadow metadata that enables its intended use cases and goals.
* **Integration:** Integrating shadow metadata system components and processes into the existing EDI technology ecosystem.
* **Presentation: **Presenting shadow metadata to the end user in a way that avoids misrepresentation of the original author's intent, clearly indicates the provenance of the shadow metadata, and explains how to interpret its veracity.
* **Maintenance:** Maintaining and supporting the above items.
* **Versioning:** Versioning and handling conflicts shadow metadata changes.


## References

## Rejection

