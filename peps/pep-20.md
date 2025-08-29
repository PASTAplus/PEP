# PEP-20: Support for Data Package Resource Thumbnail Images in PASTA <a id="top"></a>

- Author(s): Mark Servilla
- Contact: mark.servilla@gmail.com
- Status: Draft
- Type: Application
- Created: 2025-08-26
- Reviewed:
- Final:

## Table of Contents
* [Introduction](#introduction)
* [Issue Statement](#issue-statement)
* [Proposed Solution](#proposed-solution)
    * [REST API](#rest-api)
    * [EML `<additionalMetadata>`](#eml)
* [Open issue(s)](#open-issues)
* [References](#references)
* [Rejection](#rejection)
<!-- TOC -->

## Introduction <a id="introduction"></a> [^](#top)

In some cases, it would be invaluable to associate a thumbnail image with a 
data package or a data entity when displaying metadata about the data package. This PEP proposes a mechanism for associating a thumbnail image 
with a data package or data entity.

## Issue Statement <a id="issue-statement"></a> [^](#top)

A data package, including individual resources of the data package can be described by a variety of text-based metadata. However, a visual representation of the data package, and especially data entities, can be useful for users to quickly identify the composition of the data package in web pages like the Data Package landing page in the EDI Data Portal or the ezCatalog application. Unfortunately, there is no standard in the Ecological Metadata Language (EML) that allows for the specification of a thumbnail image for a data package or data entity. This PEP aims to address this issue by proposing a mechanism for associating a thumbnail image with a data package or its resources that are stored in PASTA.

## Proposed Solution <a id="proposed-solution"></a> [^](#top)

PASTA will support uploading and retrieving thumbnail images through a REST API or by declaring them in the `<additionalMetadata>` section of an EML document.

Thumbnail images will be stored in a subdirectory of the data package resource directory called `thumbnails`. Thumbnail image files will be named using a hash (TBD) value of the fully qualified URL to the data package resource it is associated with. For example, if a thumbnail image is associated with the data package entity resource `https://pasta.lternet.edu/package/data/eml/edi/100/1/23c8f9cce5a41d84ce7c2847a67070c2`, the thumbnail image will be stored in the directory `edi.100.1/thumbnails` under the name `32553441734140f869c78e41cc793a2d22003d5c.png`, which is the SHA-1 hash of the URL.

### REST API <a id="rest-api"></a> [^](#top)

The PASTA Data Package Manager REST API will include a new endpoint for uploading a thumbnail image using a multipart/form-data POST request. The endpoint will be structured as follows:
```
POST: /package/thumbnail/<resource_URL>
```
where the `<resource_URL>` is the fully qualified encoded URL to the data package resource of which the thumbnail should be associated.

To upload a thumbnail image for the above example, sending a multipart/form-data POST request to the endpoint `https://pasta.lternet.edu/package/thumbnail/https%3A%2F%2Fpasta.lternet.edu%2Fpackage%2Fdata%2Feml%2Fedi%2F100%2F1%2F23c8f9cce5a41d84ce7c2847a67070c2` with the thumbnail image embedded in the payload will associate the thumbnail image with the data package entity resource. Of course, an error will occur if the resource does not exist or if the user does not have permission to upload a thumbnail image to the resource.

Sending a GET request to the same endpoint with the same resource URL will return the thumbnail image in the response body.

### EML `<additionalMetadata>` <a id="eml"></a> [^](#top)

Alternatively, adding thumbnail information into the `<additionalMetadata>` section of an EML document will upload the thumbnail image(s) when the data package is created. This is a one-time event and cannot be used to alter or modify the thumbnail image after the data package is created (for this, use the REST API).

To add thumbnail image information into the `<additionalMetadata>` section an EML document, a `<thumbnail>` child element must be added for each thumbnail/data package resource association. The `<thumbnail>` element must have the following schema structure:

```
<thumbnail>
  <references>resource_id</references>
  <url>thumbnail_image_URL</url>
</thumbnail>
```

where the `resource_id` is the `id` attribute of the corresponding resource in the EML document of which the thumbnail should be associated. For example, if an EML document describes a data package entity resource in the following way:

```aiignore
<dataTable id="9a072c4e4af39f96f60954fc4f7d8be5">
   <entityName>leaf_litter_2014_2024.csv</entityName>
   <entityDescription>Leaf litter for Oshkosh National Forest</entityDescription>
   <physical>
...
</dataTable>
```

the `<additionalMetadata>` section of the EML document should contain the following thumbnail section:

```aiignore
<additionalMetadata>
  <describes>Thumbnail image information</describes>
  <metadata>
    <thumbnail>
      <references>9a072c4e4af39f96f60954fc4f7d8be5</references>
      <url>https://my.host.org/leaf_litter.png</url>
    </thumbnail>
  </metadata>
</additionalMetadata>
```
In the above example, the `<thumbnail>` element may be repeated for multiple thumbnail/data package resource associations.

## Open issue(s) <a id="open-issues"></a> [^](#top)

1. The REST API endpoint addition will be addressed first, followed by the EML `<additionalMetadata>` section at some point in the future.
2. Thumbnail uploads will not be supported through the EDI Data Portal desktop upload feature.

## References <a id="references"></a> [^](#top)

## Rejection <a id="rejection"></a> [^](#top)
