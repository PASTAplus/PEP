# PEP-19: API for Filtering EML Metadata

- Author(s): Colin Smith
- Contact: colin.smith@wisc.edu
- Status: Draft
- Type: Application (software)
- Created: August 20, 2025
- Reviewed: [To be determined]
- Final: [To be determined]

## Introduction

This proposal describes a new API endpoint designed to provide a lightweight and flexible method for retrieving specific metadata from Ecological Metadata Language (EML) documents. The API will perform server-side parsing and filtering to enable applications like new data catalogue search features.

## Issue Statement

Our current search strategy relies on a Solr-based search engine to index EML metadata. While effective for searching a number of pre-configured elements, this approach is limited in a couple key areas. The search engine is not configured to retrieve all possible EML elements, which restricts the depth of information that can be exposed to end users. Furthermore, the re-indexing process required to keep the search index current is an operationally burdensome and time-consuming process.

This PEP proposes a more direct, lightweight solution to complement our existing search infrastructure. The new API will allow for on-demand retrieval of any EML element via XPath queries. This provides a level of specificity and flexibility that our current search engine lacks. While this API will directly address the needs of [ezCatalog](https://github.com/EDIorg/ezCatalog), a driving factor behind its development, it is designed to serve a broader range of use cases by offering a supplement to our current search engine's capabilities.

## Proposed Solution

A new API endpoint will be created to perform server-side EML metadata parsing and filtering. This endpoint will be built using Python and the FastAPI library. It will operate on XML using XPath to ensure the highest degree of fidelity and flexibility with the source EML. An optional JSON serialization will be provided to support clients that prefer JSON over XML.

### Request Format

The API will handle a **POST** request with an `application/json` body because it avoids potential issues with complex XPath queries being included as URL parameters, such as encoding errors or URL length limitations.

Example Request Body:

```json
{
  "packageId": "edi.10.2",
  "query": [
    "/eml:eml/dataset/licensed",
    "/eml:eml/dataset/keywordSet"
  ]
}
```

1. **packageId**: A string identifier used to locate and fetch the specific EML document to be processed.  
2. **query**: A list of XPaths to filter the EML document.

The API will allow an Accept header to specify the desired response format. Supported formats will include `application/xml` and `application/json`, with `application/xml` as the default.

### Internal Workflow

Upon receiving a request, the API will perform the following steps:

1. **Receive Request:** Accept the POST request.  
2. **Retrieve Source EML:** Fetch the specified EML document, utilizing a caching layer to ensure performance. If the document is not cached, it will be retrieved from the PASTA [Read Metadata](https://pastaplus-core.readthedocs.io/en/latest/doc_tree/pasta_api/data_package_manager_api.html#read-metadata) operation.
3. **Build Intermediate XML:** Create a new intermediate XML document in memory (e.g., with an \<eml\> root) using an XML library, such as `lxml`.
4. **Execute & Aggregate:** Iterate through the query list. For each item, execute the XPath query and append the resulting XML node(s) into the intermediate XML document under the root element.
5. **Serialize Final Response:** Based on the Accept header.
6. **Return the Final Document:** Return the document in the response body.

### JSON Serialization Strategy

To manage complex, filtered XML and ensure compatibility with other EDI applications, we will implement an XML-to-JSON serialization strategy using [`Metapype`](https://github.com/PASTAplus/Metapype-eml).

1. Parsing Subset EML XML into the `Metapype` Model:  
   1. Utilize `metapype.model.metapype_io.from_xml(xml_string)` to parse the filtered EML document. This document will contain a subset of EML elements and may not be a complete, schema-valid EML record.  
   2. For accurate interpretation by `Metapype`, the root element and all other elements should use EML element names, and the structure should adhere to valid EML parent-child relationships.  
2. Exporting as JSON:
   1. Use `metapype.model.metapype_io.to_json(model)` to serialize the `Metapype` model into a JSON string.

Alternatively, if `Metapype` proves unsuitable, we will use a standard XML-to-JSON library. This library will be controlled by a centralized configuration to guarantee consistent and predictable JSON output. The configuration will define universal conversion rules, including:

* **Attribute Handling:** A consistent prefix (e.g., `_`) for all attributes.  
* **Text Value Keys:** A consistent key (e.g., `value`) for an element's text content.

### Alternatives Considered

* **Batch Processing**: Processing multiple EML documents in a single API request was considered. While this could reduce the number of API calls, it would increase the complexity of the API and could introduce a longer response time for a single request. Given that the client-side data catalog builds its data object iteratively, a single-document-per-request model is more suitable for maintaining API responsiveness and simpler client-side implementation.  
* **Expanding Solr**: Attempting to reconfigure and re-index the Solr search engine to capture all possible EML elements was considered. This was not recommended due to the significant operational burden of re-indexing and the inherent limitations of a search engine for complex, on-demand document filtering.

**Potential Negative Impacts:** The proposed solution will shift the processing load from the client to the server. While this is the intended outcome, it could result in a moderate increase in server-side resource usage. This is a justified trade-off for the improved client-side performance and overall system maintainability.

## Open issue(s)

* **Dependencies**: The implementation will require specific software dependencies for XML parsing, such as `lxml`. These need to be included in the project's dependency management file.  
* **Caching**: A strategy for server-side caching of the parsed EML documents should be considered to further improve performance for frequently requested documents. This would need to include a mechanism for cache invalidation if the source EML documents are updated.  
* **Response Schema**: The specific structure of the returned XML document, particularly the names of the wrapper element and any nested structures, needs to be finalized.

