# PEP-19: Extend `ridare` API for Filtering EML Metadata

- Author(s): Colin Smith
- Contact: colin.smith@wisc.edu
- Status: Implemented
- Type: Application (software)
- Created: August 20, 2025
- Reviewed: September 9, 2025
- Final: January 28, 2026

## Introduction

This proposal describes the extension of the [ridare](https://github.com/PASTAplus/ridare) service to provide a new endpoint for performing efficient, on-demand filtering of Ecological Metadata Language (EML) documents. By handling the complexity of XML parsing and XPath querying on the server, this API will provide any application with a simple and performant method for retrieving specific, arbitrary pieces of metadata without the overhead of downloading and processing a full EML document.

## Issue Statement

Current options for accessing EML metadata require either downloading and parsing full documents client-side, or using indexed search APIs limited to predefined fields. This creates performance and complexity burdens, especially for web clients. `ridare` already addresses some of these problems by providing server-side EML extraction and formatting, but currently lacks support for retrieving arbitrary sets of elements in a single query.

## Proposed Solution

### Functional Changes

- **Multi-element Query Support:** `ridare` will be extended to accept queries for multiple EML elements in a single call. The API will allow clients to specify a mapping of user-defined keys to XPath expressions (see below for example).

- **Flexible Response Formats:**  The `ridare` extension will support XML output. JSON output may be considered, but only if it does not introduce undue complexity. The JSON response structure and serialization strategy are specified below for reference and to facilitate discussion. An HTML output option would align with existing `ridare` functionality but may be overly complex for this use case.

- **Caching Strategy:** The service will cache source EML documents to avoid repeated downloads. Response caching of open-ended, ad hoc queries, response caching will not be implemented.

### Request Format

The API will handle a **POST** request with an `application/json` body because it avoids potential issues with complex XPath queries being included as URL parameters, such as encoding errors or URL length limitations.

Example Request Body:

```json
{
  "pid": ["edi.521.1", "knb-lter-sbc.1001.7"],  
  "query": [
    "dataset/title",  
    { "projectTitle": "dataset/project/title" }  
  ]  
}
```

1.  **pid**: An array of package identifiers used to locate and fetch the specific EML documents to be processed.
2.  **query**: An array of XPath queries or key-value pairs, where each **key** is a user-defined name for the corresponding XPath **value**. This key will be used to structure the response, allowing results to be reliably identified.

### Response Format

The response format will be XML, with JSON a possible future option subject to further discussion.

Example response (abridged):

```xml
<?xml version="1.0" encoding="utf-8"?>  
<resultset>  
  <document>  
    <packageid>edi.521.1</packageid>  
    <title>Example dataset title</title>  
    <projectTitle>  
      <title>Project title here</title>  
    </projectTitle>  
  </document>  
  <document>  
    <packageid>knb-lter-sbc.1001.7</packageid>  
    ...  
  </document>  
</resultset>
```

The API builds this document by applying a consistent set of rules based on the type of data returned by each XPath query.

* **`title`:** The query returns a single string value. This value is returned directly inside its `<title>` element.

* **`projectTitle`**: The query returns a set of `<title>` elements within a parent `<projectTitle>` element. The `projectTitle` key is used in the query to group the results. Without this grouping key, the results would be listed alongside the dataset title, and therefore indistinguishable.

* **`document`:** Each results set is grouped within a `<document>` element, which contains a `<packageid>` child element to identify the source EML document.

### Internal Workflow

Upon receiving a request, the API will perform the following steps:

1.  **Receive Request:** Accept the POST request.
2.  **Retrieve Source EML:** Fetch the specified EML document, utilizing a caching layer to ensure performance. If the document is not cached, it will be retrieved from the PASTA [Read Metadata](https://pastaplus-core.readthedocs.io/en/latest/doc_tree/pasta_api/data_package_manager_api.html#read-metadata) operation.
3.  **Build Intermediate XML:** Create a new intermediate XML document in memory (e.g., with an `<resultset>` root) using an XML library, such as `lxml`.
4.  **Execute & Aggregate:** Iterate through the key-value pairs in the `query` object. For each pair, execute the XPath query and append the resulting XML node(s) into the intermediate XML document, wrapped in an element named after the query's **key**. For simple XPath queries without a key, return the results "as is".
5.  **Serialize Final Response:** As XML.
6.  **Return the Final Document:** Return the document in the response body.

## Alternatives Considered

- **Standalone Service:** Building a new service was considered, but extending ridare is more efficient and maintainable.

- **Expanding Solr Index:** Not recommended due to operational complexity and inability to support the full richness of EML.

## Open issue(s)

