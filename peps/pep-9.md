# PEP-9: Implementing Authorization of the IAM Model
- Author(s): Mark Servilla
- Contact: mark.servilla@gmail.com
- Status: Draft
- Type: Application
- Created: 2024-11-24
- Reviewed:
- Final:


## Introduction

Authorization within the EDI application ecosystem is a critical component of the Identity and Access Management (IAM) model (see [PEP-7](./pep-7.md)). The current authorization system uses a simplified attribute-based access control (ABAC) model where an individual's identity or group membership ("group" and "role" are used interchangeably) dictates whether they are sufficiently privileged to execute a PASTA API method or to upload, access, or remove a data package, or its individual data resources (i.e., metadata, data, and quality report), from the EDI data repository. Privileges are declared in an access control rule (ACR) using the `<access>` element (Figure 1) syntax defined in the [Ecological Metadata Language](https://eml.ecoinformatics.org/schema/eml_xsd#eml_access) (EML) XML schema. The *principal* (or subject) of an access control rule is the unique user identity assigned by an Identity Provider (IdP) or an arbitrary group identifier (e.g., "authenticated") assigned by the system.  The *permissions* (or privileges) of an ACR are defined as "read," "write," or "changePermission." These map to the following: "read" to "read;" "write" collectively to "create, read, update, delete;" and "changePermission" to everything in "write," in addition to changing permissions for data resources ("all" may be substituted for "changePermission" in ACRs). These permissions are not part of the EML `<access>` element scheme; they are legacy and were defined by the Long Term Ecological Research (LTER) Network information management community.

![Figure 1](./images/pep9-EML_access_element.png)

**Figure 1:** XML schema diagram for an Ecological Metadata Language `<access>` element.

To understand an ACR, it may be read as an [RDF triple](https://www.w3.org/TR/rdf11-concepts/#section-triples), where the *principal* is the **subject**, the combination of either "allow" or "deny", along with the *permission*, is the **predicate**, and the resource under the protection of the ACR is the **object**. For example, the following `<access>` element (Listing 1) in a metadata document describing data contained in the file `table.csv` can be read informally as "the user, `uid=mark,o=EDI,dc=edirepository,dc=org`, *has permission to read* the data contained within `table.csv`." Although a "deny" verb is permitted in an `<access>` element to revoke a privilege on a resource, it is rarely used in practice. Two or more ACRs can be combined within the `<access>` element (as permitted by the XML schema) to form an Access Control List (ACL), where multiple subjects and predicates are related to a single resource object.

```xml
<access>
  <allow>
    <principal>uid=mark,o=EDI,dc=edirepository,dc=org</principal>
    <permission>read</permission>
  </allow>
</access>
```
**Listing 1:** Example of an Ecological Metadata Language `<access>` element. 

ACLs used for PASTA API methods and data resources are declared in two locations: for PASTA API methods, a standalone XML document, the `service.xml` file, contains ACRs for each method. These ACRs are read into the system during the PASTA boostrap process and can be modified as necessary by editing the file and restarting the system. ACRs for data package resources are declared in the EML metadata document describing the science data and translated into an RDBS table when first uploaded; these ACRs cannot be modified once they are read into the table. Both the `service.xml` file and the EML metadata use the same ACR structure, allowing use of the same ACR processor to determine authorization. 

This PEP proposes to improve authorization within the EDI application ecosystem by replacing IdP user identifiers and group identifiers with PASTA-based unique identifiers ([see PEP-2](./pep-2.md)) and allowing data package resource ACRs to be modified, including the addition or deletion of ACRs, anytime after the data package is initially upload and published in the EDI data repository.

## Issue Statement

The current EDI authorization system should be improved for the following reasons:

1. IdP user identifiers and group identifiers are embedded in ACRs, which are visible to the public by reviewing raw data package metadata. This potentially discloses private or sensitive information, including names, email addresses, and alternative identifiers (i.e., Orcid identifiers). These identifiers also become a permanent record in the EDI data repository audit log, which is another potential exposure route of sensitive information.
2. Data package resource ACRs become immutable once the data package is published in the EDI data repository. This poses significant hardship for data package creators or owners who wish to apply a temporary embargo to data resources during manuscript review or to modify, add, or delete ACRs when managing personnel change over time.
3. Outside the EDI data repository and the PASTA architecture, authorization processing of ACRs is not supported. This significantly limits reusability of authorization technology and the scalability of the IAM model for other EDI applications in general.

## Proposed Solution

We propose to implement a stand-alone authorization service, **AuthZ**, that will manage ACRs for all applications in the EDI ecosystem, including the EDI data repository and ezEML, and complement the new authentication service, **AuthN**, by working seamlessly with PASTA unique identifiers used for users and groups (Figure 2). AuthZ will provide a REST API for managing ACRs, including the ability to add, modify, and delete ACRs for data package resources. It will also provide a mechanism for managing ACRs for PASTA API methods. AuthZ will be designed as microservice, augmented with a web UI frontend, that integrate with the existing EDI architecture and will be implemented in Python using the FastAPI web framework.

This service will be responsible for the following:
1. Implement a secure and verifiable authorization algorithm that will process ACRs for EDI related resources.
2. Be extensible to support all applications in the EDI ecosystem.
3. Maintain a secure and private ACR registry with the necessary attributes to perform authorization based on #1.
4. Provide a REST API for managing ACRs, including the ability to add, modify, and delete ACRs.
5. Provide a web UI frontend for managing ACRs for both EDI administrators and users.

![Figure 2](./images/pep9-EDI_app_ecosystem.png)

**Figure 2:** Proposed EDI application ecosystem with the addition of the AuthZ service.

### Use Case and REST API Method Definitions

#### 1a. Add ACL

Goal: To parse a valid EML document and add its ACRs to the AuthZ ACR registry.

Use case:
1. A user uploads a data package with an EML metadata document.
2. PASTA creates a Level-1 EML metadata document and sends the EML to AuthZ to register package ACRs.
3. AuthZ parses the EML and extracts the ACRs.
4. AuthZ adds the ACRs to the ACR registry.
5. AuthZ returns a success message to PASTA.

Notes: This use case supports the existing PASTA data package upload process. Parsing and extracting ACRs from the EML document will require supporting ACRs in both the main EML document and the additional metadata section.

```http
addACL(eml: string)
    eml: valid EML document as a string
    return:
        200 OK if successful
        400 Bad Request if EML is invalid
```

#### 1b. Add ACL

Goal: To parse a valid `<access>` element and add its ACRs to the AuthZ ACR registry.

Use case:
1. An EDI application creates an `<access>` element ACL for an EDI resource.
2. The application sends the `<access>` element ACL to AuthZ to register the ACR.
3. AuthZ parses the `<access>` element ACL and extracts the ACRs.
4. AuthZ adds the ACRs to the ACR registry.
5. AuthZ returns a success message to the EDI application.

Notes: This use case supports adding ACLs for PASTA API methods through the `service.xml` file. In this case, the `service.xml` file is not a complete EML document; they consist of ACLs in the form of `<access>` elements.

```http
addACL(access: string)
    access: valid <access> element as a string
    return:
        200 OK if successful
        400 Bad Request if <access> element is invalid
```

#### 2. Add ACR

Goal: To add an individual ACR as defined by ACR attributes (TBD) to the AuthZ ACR registry.

Use case:
1. An EDI application creates an ACR for an EDI resource.
2. The application sends the ACR to AuthZ to register the ACR.
3. AuthZ validates and adds the ACR to the ACR registry.
4. AuthZ returns a success message to the EDI application.

Notes: This use case supports adding individual ACRs for applications that do not use EML or `<access>` elements.

```http
addACR(resource: string, principal: string, permission: string)
    resource: the resource to be protected by the ACR as a string
    principal: the principal of the ACR as a string
    permission: the permission of the ACR as a string
    return:
        200 OK if successful
        400 Bad Request if ACR is invalid
```

#### 3a. Is Authorized

Goal: To determine if a principal is authorized to access a resource.

Use case:
1. An EDI application collects the user's authentication token, the `<access>` element for the protected resource, and the requested permission.
2. The application sends the token, `<access>` element, and permission to AuthZ to determine if the user is authorized.
3. AuthZ processes the request and returns a success message if the user is authorized.

Notes: This use case supports the authorization process for PASTA API methods if the ACLs in `service.xml` file are not registered in AuthZ's ACR registry.

```http
isAuthorized(token: string, access: string, permission: string)
    token: a valid PASTA authentication token as a string
    access: valid <access> element as a string
    permission: the permission to be checked as a string
    return:
        200 OK if authorized
        403 Forbidden if not authorized
```

## Open issue(s)

### 1. Will AuthZ support "deny" verbs in ACRs?

Because the "deny" verb is rarely used in practice, it will not be supported in the initial implementation of the AuthZ service. However, this feature may be added in a future release.

### 2. How will the current `DataPackageManager.access_matrix` table be exported to the AuthZ service?

### 3. How will ezEML interact with the AuthZ service?

## References

...

## Rejection

...
