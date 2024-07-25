

# PEP-1: Defining Data Package Replication Using Semantic Annotation



* Author(s): Colin Smith, Mark Servilla, Greg Maurer, Kyle Zollo-Venecek
* Contact: colin.smith@wisc.edu
* Status: Draft
* Type: Policy 
* Created: 2024-05-08
* Reviewed:
* Final:


## Introduction

Ensuring data consumers are aware of replicated data across repositories is crucial.  However, the EDI Data Repository currently lacks a mechanism to achieve this. This proposal outlines a method for declaring data replication within other repositories.


## Issue Statement

The current EDI Data Repository lacks a mechanism to indicate the replication of data packages or data entities (both "data resources"), whether within the repository itself or across other repositories. This can lead to challenges:



1. **Data Consumer Confusion:** Unidentified replication can cause confusion for data consumers, hindering their ability to accurately interpret and utilize the information.

2. **Data Misuse and Errors:** Uncertainty regarding data identity (whether two resources are identical) can lead to misuse and the generation of inaccurate results.

4. **Inefficiencies for Data Harvesters**: Knowledge of replicated data allows data harvesters to optimize their processes, such as avoiding downloading duplicate copies.


## Proposed Solution

The [Ecological Metadata Language](https://eml.ecoinformatics.org/eml-ecological-metadata-language) (EML v2.2.0) offers a semantic annotation feature suitable for defining data replication, applicable to both entire data packages and specific data entities within a package. This proposal leverages this functionality alongside the [schema:sameAs](https://schema.org/sameAs) property to indicate replicated data resources. This approach aligns with [Science-On-Schema.Org](https://github.com/ESIPFed/science-on-schema.org/blob/master/guides/Dataset.md) conventions, promoting dataset discovery and interoperability.

EML provides two locations to express `schema:sameAs` relationships through annotations, within a [DatasetType](https://eml.ecoinformatics.org/schema/eml-dataset_xsd#DatasetType), and an [EntityGroup](https://eml.ecoinformatics.org/schema/eml-entity_xsd.html#EntityGroup).


### DatasetType

This location indicates relationships between data packages. Semantic RDF structures the data package as the subject, related to another package as the object. The annotation element is inserted at the XPath: `/eml/dataset/annotation`. For example:

```xml
<annotation>
      <propertyURI label="sameAs">https://schema.org/sameAs</propertyURI>
      <valueURI label="Data used to …">https://doi.org/10.5066/F7VX0DMQ</valueURI>
 </annotation>
```

Where the `valueURI` label is the dataset title and the value is the dataset Digital Object Identifier (DOI). This annotation reads: "This dataset is the same as the dataset identified by [https://doi.org/10.5066/F7VX0DMQ](https://doi.org/10.5066/F7VX0DMQ)".


### EntityGroup

This location is used to indicate relationships between data entities (e.g. dataTable or otherEntity). RDF expresses the data entity as the subject, related to another entity as the object. The annotation element is inserted within the EntityGroup. For example:

```xml
<annotation>
      <propertyURI label="sameAs">https://schema.org/sameAs</propertyURI>
      <valueURI label="black_sand_phe …">https://pasta.lternet.edu/package/data/eml/…</valueURI>
 </annotation>
```

Where the `valueURI` label is the data entity name. This annotation reads: "This data entity is the same as the entity defined by [https://pasta.lternet.edu/package/data/eml/knb-lter-nwt/237/1/39dbac0784a042fcda990797377e27ee](https://pasta.lternet.edu/package/data/eml/knb-lter-nwt/237/1/39dbac0784a042fcda990797377e27ee), which is a download link to a data entity in the EDI repository.

_One issue to be aware of here is that persistent identifiers (e.g. DOI) are typically not minted for data entities. Therefore, the resolvability of the object, depends on the stability of the link used in the annotation._


## Open issue(s)

Adopting this proposal may require adjustments to some EDI applications and processes:

1. **Data Package Best Practices:** Integrate this practice into existing best practices, outlining its usage to inform data submitters and curators.

2. **ezEML Annotation:** Develop a feature enabling the declaration of this annotation within data package metadata for user convenience.

3. **Data Portal Rendering:** Update XSLT stylesheets to display the annotation information prominently for data consumers.

4. **"Mutable Series" Consideration:** A potential challenge arises when the valueURI is a data package DOI, and pointing to the latest version within a mutable data package series. This can lead to conflicting interpretations of the relationship between data packages. For instance, if `schema:sameAs` expresses a relationship between two data packages, but a significantly different version of the object (data package in the other repository) is published, the relationship becomes inaccurate. Notably, [DataONE](https://www.dataone.org/) employs this practice for harvested data packages presented to users.