# PEP-9: Implementing Authorization of the IAM Model
- Author(s): Mark Servilla
- Contact: mark.servilla@gmail.com
- Status: Draft
- Type: Application
- Created: 2024-11-24
- Reviewed:
- Final:


## Introduction

Authorization within the EDI application ecosystem is a critical component of the Identity and Access Management (IAM) model (see [PEP-7](./pep-7.md)). The current authorization system uses a simplified attribute-based access control (ABAC) model where an individual's identity or group membership ("group" and "role" are used interchangeably) dictates whether they are sufficiently privileged to execute a PASTA API method or to upload, access, or remove a data package, or its individual data resources (i.e., metadata, data, and quality report), from the EDI data repository. Privileges are declared in access control rules (ACRs) using the `<access>` element (Figure 1) syntax defined in the [Ecological Metadata Language](https://eml.ecoinformatics.org/schema/eml_xsd#eml_access) (EML) XML schema. The *principal* (or subject) of an access control rule is the unique user identity assigned by an Identity Provider (IdP) or an arbitrary group identifier (e.g., "authenticated") assigned by the system.  The *permissions* (or privileges) of an ACR are defined as "read," "write," or "changePermission." These map to the following: "read" to "read;" "write" collectively to "create, read, update, delete;" and "changePermission" to everything in "write," in addition to changing permissions for data resources ("all" may be substituted for "changePermission" in ACRs). These permissions are not part of the EML `<access>` element scheme; they are legacy and were defined by the Long Term Ecological Research (LTER) Network information management community.

![Figure 1](./images/pep9-EML_access_element.png)

**Figure 1:** XML schema diagram for an Ecological Metadata Language `<access>` element.

To understand an ACR, it may be read as an [RDF triple](https://www.w3.org/TR/rdf11-concepts/#section-triples), where the *principal* is the **subject**, the combination of either "allow" or "deny", along with the *permission*, is the **predicate**, and the resource under the protection of the ACR is the **object**. For example, the following `<access>` element (Listing 1) in a metadata document describing data contained in the file `table.csv` can be read informally as "the user, `uid=mark,o=EDI,dc=edirepository,dc=org`, *has permission to read* the the data contained within `table.csv`." Although a "deny" verb is permitted in an `<access>` element to revoke a privilege on a resource, it is rarely used in practice. ACRs are are often combined into an Access Control List (ACL) within the `<access>` element (as permitted by the XML schema) where multiple subjects and predicates are related to a single resource object.

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
2. Data package ACRs become immutable once the data package is published in the EDI data repository. This poses significant hardship for data package creators or owners who wish to apply a temporary embargo to data resources during manuscript review or to modify, add, or delete ACRs when managing personnel change over time.
3. Outside the EDI data repository and the PASTA architecture, authorization processing of ACRs is not supported. This significantly limits reusability of authorization technology and the scalability of the IAM model for other EDI applications in general.

## Proposed Solution

...

## Open issue(s)

...

## References

...

## Rejection

...
