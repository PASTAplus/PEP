# PEP-19: API for Filtering EML Metadata

- Author(s): Colin Smith
- Contact: colin.smith@wisc.edu
- Status: Draft
- Type: Application (software)
- Created: August 20, 2025
- Reviewed: [To be determined]
- Final: [To be determined]

## Introduction

This proposal describes a new API endpoint designed to provide a more lightweight and flexible method for retrieving specific metadata from Ecological Metadata Language (EML) documents. The API will perform server-side parsing and filtering to enable applications like new data catalogue search features.

## Issue Statement

Our current search strategy relies on a Solr-based search engine to index EML metadata. While effective for searching a number of pre-configured elements, this approach is limited in a couple key areas. The search engine is not configured to retrieve all possible EML elements, which restricts the depth of information that can be exposed to end users. Furthermore, the re-indexing process required to keep the search index current is an operationally burdensome and time-consuming process.

This PEP proposes a more direct, lightweight solution to complement our existing search infrastructure. The new API will allow for on-demand retrieval of any EML element via XPath queries. This provides a level of specificity and flexibility that our current search engine lacks. While this API will directly address the needs of ezCatalog, a driving factor behind its development, it is designed to serve a broader range of use cases by offering a supplement to our current search engine's capabilities.

## Proposed Solution

A new API endpoint will be created to perform server-side EML metadata parsing and filtering. This endpoint will be built using Python and the FastAPI library.

The API will handle a **POST** request with a JSON body containing two required parameters:

1. **packageId**: A string identifier used to locate and fetch the specific EML document to be processed.  
2. **xpath\_query**: A list of strings containing one or more valid XPath query to filter the EML document.

Upon receiving a request, the API will perform the following steps:

1. Fetch the EML document identified by the packageId from its source.  
2. Parse the EML document using an efficient XML library, such as lxml.  
3. Apply the list of xpath\_query to the parsed document to select the desired metadata elements.  
4. Construct a new, well-formed XML document from the filtered elements. This new document will use a generic root element, such as \<document\>, to ensure it is a valid XML structure.  
5. Return the final XML document in the response body.

This design offloads the parsing and filtering from the client, simplifying the client-side data aggregation logic. The use of a **POST** request is recommended because it avoids potential issues with complex XPath queries being included as URL parameters, such as encoding errors or URL length limitations. The API will be designed to process one EML document per request, which simplifies the server-side logic and aligns well with the iterative data aggregation process on the client side.

**Alternatives Considered:**

* **Batch Processing**: Processing multiple EML documents in a single API request was considered. While this could reduce the number of API calls, it would increase the complexity of the API and could introduce a longer response time for a single request. Given that the client-side data catalog builds its data object iteratively, a single-document-per-request model is more suitable for maintaining API responsiveness and simpler client-side implementation.  
* **Expanding Solr**: Attempting to reconfigure and re-index the Solr search engine to capture all possible EML elements was considered. This was not recommended due to the significant operational burden of re-indexing and the inherent limitations of a search engine for complex, on-demand document filtering.

**Potential Negative Impacts:** The proposed solution will shift the processing load from the client to the server. While this is the intended outcome, it could result in a moderate increase in server-side resource usage. This is a justified trade-off for the improved client-side performance and overall system maintainability.

## Open issue(s)

* **Dependencies**: The implementation will require specific software dependencies for XML parsing, such as lxml. These need to be included in the project's dependency management file.  
* **Caching**: A strategy for server-side caching of the parsed EML documents should be considered to further improve performance for frequently requested documents. This would need to include a mechanism for cache invalidation if the source EML documents are updated.  
* **Response Schema**: The specific structure of the returned XML document, particularly the names of the wrapper element and any nested structures, needs to be finalized.

