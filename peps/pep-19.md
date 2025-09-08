# PEP-19: API for Filtering EML Metadata

- Author(s): Colin Smith
- Contact: colin.smith@wisc.edu
- Status: Draft
- Type: Application (software)
- Created: August 20, 2025
- Reviewed: [To be determined]
- Final: [To be determined]

## Introduction

This proposal describes a new, general-purpose API endpoint for performing efficient, on-demand filtering of Ecological Metadata Language (EML) documents. This service is designed to be a new tool in the EDI data ecosystem, complementing existing services like the [`Read Metadata`](https://pastaplus-core.readthedocs.io/en/latest/doc_tree/pasta_api/data_package_manager_api.html#read-metadata) and [`Search Data Packages`](https://pastaplus-core.readthedocs.io/en/latest/doc_tree/pasta_api/data_package_manager_api.html#search-data-packages) APIs.

By handling the complexity of XML parsing and XPath querying on the server, this API will provide any application with a simple and performant method for retrieving specific, arbitrary pieces of metadata without the overhead of downloading and processing a full EML document.

## Issue Statement

Currently, developers have two primary options for accessing EML metadata: retrieving a full document via the `Read Metadata` endpoint or using the `Search Data Packages` API. While powerful, they represent two ends of a spectrum, leaving a gap for applications that need targeted, flexible data retrieval.

The `Search Data Packages` API is fast but limited to a predefined set of indexed fields, making it unsuitable for accessing the full richness of the EML. On the other hand, the `Read Metadata` endpoint provides the complete document but forces the burden of processing onto the client. Requiring every client application to download, parse, and filter large XML documents is inefficient and presents several disadvantages:

* **Performance:** It creates significant network overhead and demands high CPU and memory resources on the client side, which is especially problematic for web and mobile applications.  
* **Complexity:** It requires every developer to implement their own robust XML parsing and XPath logic, increasing development time and the potential for inconsistencies.  
* **Scalability:** It makes simple tasks, like fetching a list of creator names from multiple data packages, a cumbersome, multi-step process for the client.

This proposal addresses this gap by providing a server-side solution that combines the flexibility of direct document querying with the efficiency of a lightweight data transfer.

## Proposed Solution

A new API endpoint will be created to perform server-side EML metadata parsing and filtering. This endpoint will be built using Python and the FastAPI library. It will operate on XML using XPath to ensure the highest degree of fidelity and flexibility with the source EML. An optional JSON serialization will be provided to support clients that prefer JSON over XML.

### Request Format

The API will handle a **POST** request with an `application/json` body because it avoids potential issues with complex XPath queries being included as URL parameters, such as encoding errors or URL length limitations.

Example Request Body:

```json
{
   "packageId": "edi.2114.1",
   "query": {
      "abstract": "string(/eml:eml/dataset/abstract)",
      "keywords": "/eml:eml/dataset/keywordSet/keyword/text()",
      "creators": "/eml:eml/dataset/creator"
   }
}
```

1.  **packageId**: A string identifier used to locate and fetch the specific EML document to be processed.
2.  **query**: An object where each **key** is a user-defined name for the corresponding XPath **value**. This key will be used to structure the response, allowing results to be reliably identified.

### Response Format

The API will allow an `Accept` header to specify the desired response format. Supported formats will include `application/json` and `application/xml`, with `application/json` as the default.

#### XML Response Example

If `application/xml` is requested, the response will be an XML document structured as follows:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<results>
  <abstract>This dataset, lakeSR: AquaMatch Landsat Collection 2 surface ...</abstract>
  <keywords>
    <item>aquatic remote sensing</item>
    <item>surface temperature</item>
    <item>reflectance</item>
    <item>temperature</item>
    <item>landsat</item>
  </keywords>
  <creators>
    <creator>
      <individualName>
        <givenName>Bethel</givenName>
        <givenName>G</givenName>
        <surName>Steele</surName>
      </individualName>
      <organizationName>Colorado State University</organizationName>
      <electronicMailAddress>b.steele@colostate.edu</electronicMailAddress>
      <userId directory="https://orcid.org">0000-0003-4365-4103</userId>
    </creator>
    <creator>
      <individualName>
        <givenName>Matthew</givenName>
        <givenName>R</givenName>
        <surName>Brousil</surName>
      </individualName>
      <organizationName>Colorado State University</organizationName>
      <electronicMailAddress>matthew.brousil@colostate.edu</electronicMailAddress>
      <userId directory="https://orcid.org">0000-0001-8229-9445</userId>
    </creator>
  </creators>
</results>
```

The API builds this document by applying a consistent set of rules based on the type of data returned by each XPath query.

* **`abstract`:** The query returns a single string value. This value is placed directly inside a new `<abstract>` element, named after the key.

* **`keywords`**: The query returns a list of text values. To represent this list, a parent `<keywords>` element is created. Each individual text value is then placed in a generic child element, like `<item>`.

* **`creators`:** The query returns a set of existing, complex XML nodes. A parent `<creators>` element is created to hold them, and the original `<creator>` nodes are appended inside it, preserving their full structure and fidelity.

#### JSON Response Example

If `application/json` is requested, the response will be a JSON object structured as follows:

```json
{
  "abstract": "This dataset, lakeSR: AquaMatch Landsat Collection 2 surface ...",
  "keywords": [
    "aquatic remote sensing",
    "surface temperature",
    "reflectance",
    "temperature",
    "landsat"
  ],
  "creators": [
    {
      "individualName": {
        "givenName": [
          "Bethel",
          "G"
        ],
        "surName": "Steele"
      },
      "organizationName": "Colorado State University",
      "electronicMailAddress": "b.steele@colostate.edu",
      "userId": {
        "_directory": "https://orcid.org",
        "value": "0000-0003-4365-4103"
      }
    },
    {
      "individualName": {
        "givenName": [
          "Matthew",
          "R"
        ],
        "surName": "Brousil"
      },
      "organizationName": "Colorado State University",
      "electronicMailAddress": "matthew.brousil@colostate.edu",
      "userId": {
        "_directory": "https://orcid.org",
        "value": "0000-0001-8229-9445"
      }
    }
  ]
}
```

This JSON structure is the result of applying consistent serialization rules (described below). The goal is to produce a format that is easy for client-side applications to consume.

* **`abstract`:** A simple XML element with text becomes a standard **key-value string**.  
* **`keywords`:** An element containing a list of simple child elements (`<item>`) is converted into a **JSON array of strings**.
* **`creators`:** An element containing a list of complex child elements (`<creator>`) is converted into a **JSON array of objects**.  
  * Within each creator object, the repeated `<givenName>` tag is converted into an **array of strings**.  
  * The `<userId>` element, which has both an attribute and a text value, is converted into an **object**, with the attribute prefixed (e.g., `_directory`) and the text content given a standard key (e.g., `value`).

### Internal Workflow

Upon receiving a request, the API will perform the following steps:

1.  **Receive Request:** Accept the POST request.
2.  **Retrieve Source EML:** Fetch the specified EML document, utilizing a caching layer to ensure performance. If the document is not cached, it will be retrieved from the PASTA [Read Metadata](https://pastaplus-core.readthedocs.io/en/latest/doc_tree/pasta_api/data_package_manager_api.html#read-metadata) operation.
3.  **Build Intermediate XML:** Create a new intermediate XML document in memory (e.g., with an `<results>` root) using an XML library, such as `lxml`.
4.  **Execute & Aggregate:** Iterate through the key-value pairs in the `query` object. For each pair, execute the XPath query and append the resulting XML node(s) into the intermediate XML document, wrapped in an element named after the query's **key**.
5.  **Serialize Final Response:** Based on the `Accept` header.
6.  **Return the Final Document:** Return the document in the response body.

### JSON Serialization Strategy

To handle the conversion of filtered XML, our strategy is to use a standard XML-to-JSON serialization library controlled by a strict, centralized configuration. This ensures the resulting JSON is predictable, clean, and easy for client applications to consume.

The configuration will enforce a consistent set of universal rules.

  * **Attribute Handling:** A consistent prefix (e.g., `_`) will be used for all attributes to distinguish them from elements.
  * **Text Value Keys:** A consistent key (e.g., `value`) will be used for an element's text content when it also has attributes.
  * **Forced Arrays:** A list of element names that can appear more than once will be configured to always produce a JSON array, even if the query returns only a single instance. This prevents data type inconsistencies.

For consistency across the broader EDI software ecosystem, our serialization configuration should align as closely as possible with the conventions used by `Metapype`'s own XML-to-JSON output.

## Alternatives Considered

* **Status Quo: Relying on Client-Side Filtering**: The primary alternative is to not build this service and require clients to continue using the existing `Read Metadata` endpoint for custom filtering. This was not recommended because it forces significant network and processing loads onto the client and requires every application to build and maintain its own complex XML parsing solution.  
* **Expanding Solr**: Attempting to reconfigure and re-index the Solr search engine to capture all relevant EML elements was considered. This was not recommended due to the significant operational burden of re-indexing and the inherent limitations of a search engine for performing complex, on-demand document filtering with the precision of XPath.  
* **Batch Processing**: Processing multiple EML documents in a single API request was considered. While this could reduce the number of API calls, it would increase the complexity of the API. Given that many client-side applications build their data objects iteratively, a single-document-per-request model is more suitable for maintaining API responsiveness.

## Open issue(s)

* **Dependencies**: The implementation will require specific software dependencies for XML parsing, such as `lxml`. These need to be included in the project's dependency management file.
* **Caching**: A strategy for server-side caching of EML documents should be considered to further improve performance for frequently requested documents. This would need to include a mechanism for cache invalidation if the source documents are updated.
* **API Documentation**: The public-facing API documentation needs to be created. It must clearly detail:
    * The specific **XPath version** supported by the service (e.g., XPath 1.0) so clients can construct valid queries.
    * The rules of the **JSON Serialization Strategy**, including attribute handling and forced arrays, so clients can reliably parse the JSON response.
