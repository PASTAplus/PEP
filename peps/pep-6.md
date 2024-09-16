# PEP-6: Displaying EML Annotations in the EDI Data Portal

- Author(s): Colin Smith
- Contact: colin.smith@wisc.edu
- Status: Draft
- Type: Application
- Created: 2024-09-16
- Reviewed:
- Final:


## Introduction

Currently, the EDI data portal has limited functionality for displaying EML annotations. Additionally, the existing display pattern tries to prescribe meaning to the annotation before it is known. This PEP proposes a comprehensive pattern for displaying EML annotations with the flexibility to expand incrementally.

## Issue Statement

The EDI data portal currently only displays EML annotations for data entity attributes. However, the EML schema includes many other elements that can be annotated and that could be supported in the future. Some of these elements are already being used, or will soon be adopted, by data authors. Additionally, the current display format assigns meaning to annotations through a field labeled "External Measurement Definition, Link," which presumes a property value of `containsMeasurementsOfType`. This can create confusion when attribute annotations have different property values, such as `has unit`, which conflicts with the implied meaning of the field.

This PEP proposes a comprehensive, modular, and extensible approach to rendering EML annotations for data packages in the EDI data portal. It recommends immediate support for elements that are currently in use, while allowing for future expansion to other elements as needed.

Adopting this PEP will benefit both EDI and the broader user community. Annotations offer precise definitions of metadata concepts, and when displayed intuitively, they enhance data comprehension and allow for quicker assessments of how a data package can be used in research. Furthermore, this PEP supports data authors already publishing packages with annotations and advances EDI’s efforts to display key information, such as [data package replication](https://github.com/PASTAplus/PEP/blob/main/peps/pep-1.md). As the community develops shared practices for metadata annotation, we anticipate an increase in both annotation usage and the importance of proper display.


## Proposed Solution

### Selecting Elements to Support

There are five locations within the [EML 2.2.0 schema](https://eml.ecoinformatics.org/) where annotations can be included:
* top-level resource (e.g. dataset)
* entity-level (e.g. dataTable)
* attribute 
* eml/annotations
* eml/additionalMetadata

The eml/annotations container enables referencing of a main-body element by its id attribute thereby expanding the list of elements that can be annotated considerably. Of this long list of elements, we propose supporting the known subset data authors are already using and those we anticipate being the most immediately useful, which include:
* **top-level resource:** DatasetType
* **entity-level:** DataTableType, OtherEntityType, SpatialRasterType SpatialVectorType, StoredProcedureType, ViewType
* **attribute:** AttributeType

This list can be expanded as the community demonstrates need through usage.

### Rendering Annotation Elements

Each annotation is made up of a `propertyURI` and a `valueURI`, both of which have human-readable labels and machine-resolvable values. We propose rendering the human-readable labels in a sentence form and hyperlinking them with their corresponding URI values. Furthermore, we recommend underlining each property and value label to group blank space separated words for readability. For example the EML annotation:

```
<annotation>
    <propertyURI label="containsMeasurementsOfType">http://ecoinformatics.org/oboe/oboe.1.2/oboe-core.owl#containsMeasurementsOfType</propertyURI>
    <valueURI label="Conductivity">http://ecoinformatics.org/oboe/oboe.1.2/oboe-characteristics.owl#Conductivity</valueURI>
</annotation>
```

can be rendered as:

<u>[containsMeasurementsOfType](http://ecoinformatics.org/oboe/oboe.1.2/oboe-core.owl#containsMeasurementsOfType)</u> <u>[Conductivity](http://ecoinformatics.org/oboe/oboe.1.2/oboe-characteristics.owl#Conductivity)</u>

Since multiple annotations can exist for a single EML element, each annotation would be displayed in the format described above, separated by commas and new lines to facilitate readability. For example:

```
<annotation>
    <propertyURI label="containsMeasurementsOfType">http://ecoinformatics.org/oboe/oboe.1.2/oboe-core.owl#containsMeasurementsOfType</propertyURI>
    <valueURI label="Conductivity">http://ecoinformatics.org/oboe/oboe.1.2/oboe-characteristics.owl#Conductivity</valueURI>
</annotation>
<annotation>
    <propertyURI label="has unit">http://qudt.org/schema/qudt/hasUnit</propertyURI>
    <valueURI label="Microsiemens per Meter">http://qudt.org/vocab/unit/MicroS-PER-M</valueURI>
</annotation>
```

can be rendered as:

<u>[containsMeasurementsOfType](http://ecoinformatics.org/oboe/oboe.1.2/oboe-core.owl#containsMeasurementsOfType)</u> <u>[Conductivity](http://ecoinformatics.org/oboe/oboe.1.2/oboe-characteristics.owl#Conductivity)</u>,

<u>[has unit](http://qudt.org/schema/qudt/hasUnit)</u> <u>[Microsiemens per Meter](http://qudt.org/vocab/unit/MicroS-PER-M)</u>

### Grouping Annotations

We recommend introducing a new "Annotation" field in the data portal to group these annotations, much like existing metadata fields. Since annotations can vary widely, a generic "Annotation" field avoids complications that might arise from prescribing their specific meaning. For example:

![image info](/peps/images/annotation_pattern.png)

This general pattern can be applied to group annotations near the related metadata in the portal.

### Locating Annotation Displays

Our recommended approach is to display annotations near their corresponding metadata elements in the full metadata page of a data package. This close proximity creates an intuitive connection between the metadata and its annotations. The flexibility of this method allows for incremental implementation of new annotations as their usage grows within the community.

We currently avoid displaying annotations on the data package summary page to keep the content high-level and free from potentially lengthy lists of annotations. However, this approach does not rule out the possibility of adding annotations to the summary page in the future if certain annotations are deemed valuable enough to display there.

Below are mockups demonstrating the display of annotations for the elements we propose supporting:

**Dataset**

![image info](/peps/images/dataset_annotation_display.png)

**DataEntity**

![image info](/peps/images/data_entity_annotation_display.png)

**Attribute**

![image info](/peps/images/attribute_annotation_display.png)

## Open issue(s)

1. **Update XSLT Stylesheet:** The [EML XSLT stylesheet](https://github.com/PASTAplus/DataPortal/blob/277f8230d939d4a30268d3ca9b311576b2f8f074/WebRoot/WEB-INF/xsl/eml-2.xsl) must be updated to render annotations for the above elements in the proposed format.
