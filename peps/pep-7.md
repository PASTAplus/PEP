# PEP-7: Upgrading the EDI Identity and Access Management (IAM) Model

Author(s): Mark Servilla  
Contact: [mark.servilla@gmail.com](mailto:mark.servilla@gmail.com)	  
Status: Draft  
Type: Application  
Created: 2024-08-27  
Reviewed:  
Final:

# Introduction

The Environmental Data Initiative (EDI) proposes to upgrade its Identity and Access Management (IAM) model across all its applications. Identity and access management is a framework of policies and technologies ensuring that only users who have proven their identity and with appropriate permissions can access repository services and data resources. For years, EDI has relied on a local LDAP user registry as its internal identity provider (IdP) for users who need to archive and publish scientific data through the EDI data repository. More recently, EDI has embraced single sign-on (SSO) authentication using external IdPs, providing users with more straightforward options for identity verification, albeit with fewer privileges in EDI applications. Although this approach has worked well over the years, EDI believes that updating the IAM model will significantly improve usability and security throughout the EDI application ecosystem in the following areas:

**User management**. Integrating user-managed EDI profiles will streamline how EDI software applications identifies and interacts with users (e.g., notifications) based on information held within their profile.

**Group management**. Allowing users to create and manage their onwn groups will reduce the time and complexitity required to manage access control rules (ACRs) for individual users.

**Identity mapping**. Mapping external IdP identities to a single user profile will allow users to authenticate with different IdPs, but use the same EDI "user profile" identity within the EDI application ecosystem.

**JSON Web Tokens**. JSON Web Tokens (JWTs) are an industry recognized authentication token that is supported by many different SDK libraries and tools, simplifying the process to build and parse tokens.

**Programmable Access Control**. Programmable access control rules will allow administrators and users to create and modify ACRs for resources dynamically, without pre-specifying ACRs in static form (e.g., EML document, service file, or software).

**Accept deprecation of EML 2.2.0 `<access>` element**. Pivot from using the EML `<access>` element from within the main body to recognizing them in the schema-free `<additionalMetadata>` element of EML, thereby preserving the ability to pre-write ACRs within the EML document.
 
# Background

There are currently five EDI applications that use some form of IAM:

1. **Authentication Service** - Performs user authentication through one internal IdP and four external Oauth-IdPs (see Table 1 below), returning an authentication token to the client requesting user authentication.
2. **Core data repostiory (PASTA)** - Consumes identity credentials in exchange for an authentication token from the Authentication Service and performs internal authorization for repository web-service REST API methods and data package resources.
3. **Data Portal** - Consumes identity credentials in exchange for an authentication token from the Authentication Service or redirects to the Authentication Service for Oauth authentication and stores the returned authentication tokens for future interactions with the data repository, while privileged identities are allowed to execute restricted functionality in the web application. 
4. **ezEML** - Consumes identity credentials in exchange for an authentication token from the Authentication Service or redirects to the Authentication Service for Oauth authentication and stores the unique user identity extracted from the authentication token for authorization to local metadata resources.
5. **Dashboard** - Performs local user authentication through internal (LDAP) IdP by way of an LDAP bind for authorization to restricted functionality in the web application.

EDI uses five different identity providers (IdPs) to authenticate users (Table 1). Upon successful authentication, each IdP returns a unique identifier that is used within the authentication token (see below) to uniquely identify the user. The core data repostiory (PASTA), Data Portal, and ezEML all rely on the Authentication Service to perform user authentication. Only the Dashboard performs user authentication directly to the EDI LDAP IdP, by-passing the Authentication Service completely.

| Identity Provider | Unique Identifier  | Example                                            |
|-------------------|--------------------|----------------------------------------------------|
| EDI LDAP          | Distinguished Name | uid=mark,o=EDI,dc=edirepository,dc=org             |
| GitHub            | Namespace          | <span>https://</span>github.com/mark               |
| Google            | Email              | mark<span>@</span>gmail.com                        |
| Microsoft         | Unique Identifier  | wdKzhHw0bxfW4dT5RNhpXz0h-s7NGR2K54155VI0Wpk        |
| Orcid             | Orcid Identifier   | <span>https://</span>orcid.org/0005-0327-2290-9230 |

**Table 1**: Identity providers and the different types of unique user identifiers returned by each.

The authentication token (Listing 1) is a non-standard string containing information about the user's identity, including: 1) unique identifier, 2) secutity namespace, 3) token time-to-live, and 4) groups the use may belong to. Each field is separated by an asterisk "*"; all fields, except for groups, are required.

```
mark@gmail.com*https://pasta.edirepository.org/authentication*1531891534443*authenticated
```
**Listing 1**: Example of an EDI authentication token with fields separated by "*" asterisks. Fields are ordered by user identifier, system namespace, time-to-live, and groups.

Authorization within EDI applications take on many forms. The data repository, which uses the PASTA software, has the most comprehensive use of access control within EDI: access control rules (ACRs), in the form of EML `<access>` elements, provide the basis for controlling access to both REST API methods and data package resources. These ACRs define who may access the resource and at what level of access (e.g., "read", "write", and "change permission"). For the REST API methods, ACRs are parsed and stored in memory while the system is running. Data package resource ACRs, on the other hand, are parsed from the EML metadata in which they are declared and stored in a relational database table for easy and quick processing of the rule when the resource is requested. This database table, labeled the `access_matrix`, stores the pertinenet information to determine whether a user can access a data package resource, including the user's unique IdP identifier as declared in the ACR, the allowed permission, and the fully-qualified identifier of the resource (i.e., the persistent URI of the resource). Enforcing access control in the data repository is multi-tiered: a user must first be allowed to execute the REST API method required to access the data package resource, followed by having sufficient rights to access the data resource at the level they require. If either level of access is not met, the request is denied. It is mentionable that the PASTA software used by the data repository denies access to all resources, including REST API methods, from users unless there is an explicit ACR allowing access.

# Issue Statement

EDI’s IAM model works as expected but can be improved. Its original design was intended for a single community of users, the LTER Network. A locally managed LDAP registry, along with LDAP "distinguished names," was the only framework needed to achieve authentication and authorization goals. Years later, single sign-on (SSO) using third-party identity providers was added to the framework to support LTER sites wishing to track consumers of their data products more effectively. Today, the different identity providers and the simple use of their unique identifiers in the authentication token limits EDI’s ability to serve a broad community user base. The issues follow:

**1. Inconsistent format of unique identities**

The different modalities used for unique identifiers returned by each identity provider complicate matching users with access control rules in several ways. First, users need to know the exact identifier in the authentication token, which must be added into the access control rule of the EML `<access>` element. This is error-prone since the identifier string in both locations must be identical. Moreover, Orcid, and especially Microsoft, identifiers consist of alpha-numeric values that are more difficult to copy and use. To a lesser concern, the unique identifiers are visible to users and expose a level of personal information in the EML metadata and the event logs of EDI’s audit service. 

**2. Inability to create, update, or delete user-owned groups.**

Although the IAM model supports the assignment and evaluation of user groups, the EDI data repository does not allow users to create or manage groups directly. Groups can simplify the creation of access control rules for resource authorization by consolidating sets of users into groups.

**3. Inability to use different identities with a single EDI account.**

Users cannot use multiple identity providers to sign in to the EDI ecosystem and be recognized as the same user for resource authorization. Users must always sign in with the EDI LDAP identity provider when data repository actions require membership in the "vetted" group, even if another identity provider authenticates their identity. In ezEML, this creates numerous user spaces, one for each unique identifier, where work is saved during EML editing sessions. This leads to confusion when users sign in with different identity providers and unintentionally spread their work across different user spaces.

**4. Authentication tokens cannot easily scale.**

Dependence on conveying identity information from the authentication service to clients and the authorization service relies on the structure and scalability of the PASTA authentication token. This token (Listing 1) is based on an ordered list of values, meaning that positional placement is critical to interpreting each value. The lack of a more flexible and scalable approach, like key-value pairs, limits our ability to easily scale the use of the authentication token within the EDI ecosystem for new authentication and authorization models.

**5. Inability to create, update, or delete user-defined access control rules.**

For data resources, all access control rules must be in EML metadata `<access>` elements before publishing the data package. As a result, the user cannot change or modify access rules after the data package is published. Scenarios where users require temporary embargoes on data when publishing an associated journal article or if new users require update privileges to a data package after publication are not possible in the EDI ecosystem.

**6. Deprecation of the \<access\> element in EML 2.2.0.**

The EML 2.2.0 standard used by the EDI data repository and ezEML has officially deprecated the EML metadata `<access>` element. This XML element will eventually be removed from the schema, resulting in validation errors if not removed from the EML metadata document. EDI must provide an alternative mechanism to meet the goal of data resource authorization.

# Proposed Solution

The EDI software development team proposes a multi-faceted solution to ensure the goals of the IAM model are met while expanding the user and group management features provided to end users. Six areas of the IAM model will be upgraded:

**1. User management**

We will introduce a new user profile object to capture the salient information required of a user (e.g., common name, email address, or notification preferences). This profile object will be created upon the first authentication of a user for each distinct identity provider. It will be intrinsically tied to that identity provider through the unique identifier. It will receive a unique identifier (PASTA ID) relevant only to applications of the EDI ecosystem. The PASTA ID will supplant the use of the unique identifiers returned by the identity providers for resource authorization, meaning that PASTA IDs will replace the identity provider's unique identifiers that go into access control rules. The authentication tokens will no longer contain the unique identifier. Instead, the authentication service will insert the PASTA ID into the authentication token for use with resource authorization. This will unify the access control RDBMS table identifiers and eliminate direct exposure of the personal information visible in the identity provider’s unique identifiers (Table 3). We will let users remove their user profile objects, effectively removing them from the EDI ecosystem when all user profile objects of a user no longer exist.

See this EDI PEP for more information about the user profile and PASTA ID in the EDI ecosystem: https://github.com/PASTAplus/PEP/blob/main/peps/pep-2.md.

| Data Resource                                                                                       | Identity                         | Access Type | Access Order | Permission |
|-----------------------------------------------------------------------------------------------------|----------------------------------|-------------|--------------|------------|
| <span>https://</span>pasta.lternet.edu/package/data/eml/edi/1220/6/79e0ef272ea569ae12a531306bda59fd | PASTA-a8809d422e455f9843b9024ed4 | allow       | allowFirst   | write      |
| <span>https://</span>pasta.lternet.edu/package/data/eml/edi/1220/6/f65f76748fcbbfdac1d48a476ae86794 | PASTA-a8809d422e455f9843b9024ed4 | allow       | allowFirst   | write      |
| <span>https://</span>pasta.lternet.edu/package/data/eml/edi/1220/6/d58ab68c88a86a28fc5e46bf05f7edfb | PASTA-a8809d422e455f9843b9024ed4 | allow       | allowFirst   | write      |
| <span>https://</span>pasta.lternet.edu/package/metadata/eml/edi/1220/6                              | PASTA-a8809d422e455f9843b9024ed4 | allow       | allowFirst   | write      |
| <span>https://</span>pasta.lternet.edu/package/report/eml/edi/1220/6                                | PASTA-a8809d422e455f9843b9024ed4 | allow       | allowFirst   | write      |
| <span>https://</span>pasta.lternet.edu/package/eml/edi/1220/6                                       | PASTA-a8809d422e455f9843b9024ed4 | allow       | allowFirst   | write      |
| <span>https://</span>pasta.lternet.edu/package/report/eml/edi/1220/6                                | PASTA-82c934a09235d8903249b8cd92 | allow       | allowFirst   | read       |
| <span>https://</span>pasta.lternet.edu/package/metadata/eml/edi/1220/6                              | PASTA-82c934a09235d8903249b8cd92 | allow       | allowFirst   | read       |
| <span>https://</span>pasta.lternet.edu/package/eml/edi/1220/6                                       | PASTA-82c934a09235d8903249b8cd92 | allow       | allowFirst   | read       |
| <span>https://</span>pasta.lternet.edu/package/data/eml/edi/1220/6/f65f76748fcbbfdac1d48a476ae86794 | PASTA-82c934a09235d8903249b8cd92 | allow       | allowFirst   | read       |
| <span>https://</span>pasta.lternet.edu/package/data/eml/edi/1220/6/d58ab68c88a86a28fc5e46bf05f7edfb | PASTA-82c934a09235d8903249b8cd92 | allow       | allowFirst   | read       |
| <span>https://</span>pasta.lternet.edu/package/metadata/eml/edi/1220/6                              | PASTA-65a821324c98234d98238a5559 | allow       | allowFirst   | all        |
| <span>https://</span>pasta.lternet.edu/package/report/eml/edi/1220/6                                | PASTA-65a821324c98234d98238a5559 | allow       | allowFirst   | all        |
| <span>https://</span>pasta.lternet.edu/package/eml/edi/1220/6                                       | PASTA-65a821324c98234d98238a5559 | allow       | allowFirst   | all        |
**Table 3**: A snippet of the RDBMS table showing access control rules for data resources, but with unified identities that do not disclose personal information. Note that column definitions are identical to Table 1.

**2. Group management**

We will introduce a new group object fully managed by users, allowing them to create, modify, and delete group objects. Like the user profile's PASTA ID, each group object will also receive a PASTA ID, which can also be used in access control rules. Groups will be owned by their creators and have a human-readable name similar to a user’s common name.  However, the group object owner may transfer ownership to another valid user. Groups can only contain users with a valid user profile.

Similar to other repository resources, groups will also be managed as an access-controlled resource. The group creator, as **OWNER**, will have full rights, including the privilege to assign the following permissions to other users in the group:

- **READ** - to see others in the group,
- **WRITE** - to add or delete other users in/from the group, and
- **OWNER** - to modify the permissions of another user in the group and to delete the group (group deletion may occur even if members exist).

A group owner may change their permission, including removing themself from the group, only if at least one other member has the **OWNER** permission.

**3. Identity mapping**

We will provide a mechanism by which a user with at least one valid user profile can link unique identifiers from other identity providers to that profile, thereby recognizing the same user in the EDI ecosystem regardless of their authentication pathway (Figure 2). This process will require first signing in with the identity provider used to create the target profile, then signing in with another identity provider, at which point the two unique identifiers would map to the same target profile. If the newly linked identifier is already associated with another user profile, that other profile will be removed during the mapping process, and any access control rules associated with the now defunct profile will be reassigned with the target profile PASTA ID. We will also allow a user to "unlink" an identifier from a profile. This will enable the user to create a new user profile from the unlinked identifier when the user signs in again with that identity provider.

![Figure 2](./images/pep7-nomapped_and_mapped_identities.png){ width=75% }

**Figure 2**: Conceptual comparison between the IAM model without identity mapping and with identity mapping. Using unique identifiers creates multiple personas in the EDI ecosystem, leading to confusion with access control rules and how user content is stored in applications like ezEML. In contrast, user profiles and identity mapping allow the EDI ecosystem to recognize "Mark Sidari" as a single person based on a single profile identifier, eliminating confusion. 

**4. Transitioning to JSON Web Tokens (JWT)**

We will replace the PASTA authentication token with a JSON Web Token (JWT). JWTs are an industry standard recognized by the JWT specification, RFC 7519. Since the JWT is a JSON data structure, values are defined using key-value pair notation, eliminating the ambiguity found with the positional values of the PASTA authentication token. JWTs have a standard set of key-value pairs called "registered claims," including definitions for identifying the identity provider, user, audience, and the token time-to-live. JWTs also have "private claims," allowing EDI to add key-value pairs specific to our needs. The following (Listing 3) is an example of a JWT payload:

```json
{
    "iss": "https://authn.edirepository.org",
    "sub": "PASTA-d8e8ba7d848141b3a864cfc6daf97b89",
    "at_hash": "HK6E-P6Dh8y93mRNtsDB1Q",
    "email": "mark@gmail.com",
    "email_verified": "true",
    "iat": 1353601026,
    "exp": 135e604926,
    "hd": "edirepository.org",
    "idp": "google.com",
    "uid": "mark@gmail.com",
    "gn": "Mark",
    "sn": "Sidari",
    "cn": "Mark Sidari"
}
```
**Listing 3**: Example JSON web token payload that could be used as a replacement for the PASTA authentication token. Lines 2 through 10 are "registered claims," and 11 through 15 are "private claims."

An additional benefit of JWTs is that they avoid issues with Cross-Origin Resource Sharing (CORS) restrictions when sent through the HTTP Authorization header as a "Bearer" token.

See this EDI PEP for more information about JSON Web Tokens in the EDI ecosystem: https://github.com/PASTAplus/PEP/blob/main/peps/pep-3.md.

**5. User-managed access control rules**

We will provide a mechanism by which the owner of a data resource or any user who has permission to modify access control rules of a data resource can create, modify, and delete access control rules for that data resource post-publication of the data package. This mechanism will allow that user to search for other users with a valid user profile or a group owned by a user with a valid user profile and create a new access control rule that will assign permission to "read," "update," or "all/change_permissions" to that data resource using the PASTA IDs of the other users or groups, respectively. This mechanism will also allow that user to modify the permissions of an existing access control rule for a data resource or delete the access control rule for a data resource. The benefit of allowing post-publication user-managed access control rules is that users can decide who and when access should be permitted to their data resources.

**6. Addressing deprecation of EML 2.2.0 `<access>` element**

We will modify the process by which EML 2.2.0 (or greater) parses and extracts access control rules from the EML metadata `<access>` element to one that identifies similar access control rule schemas within the `<additionalMetadata>` element of the EML document (the `<additionalMetadata>` element of the EML metadata schema permits any valid XML content that is not within the EML schema namespace). We will continue to extract access control rules using the existing `<access>` elements of the EML metadata for EML versions before and up to 2.2.0, but be ready for versions greater than 2.2.0 if the `<access>` element is no longer schema-valid. We believe access control rules within the EML metadata are important for scenarios where users would like a default set of access control rules added as part of the publication workflow.

# Open issue(s)

**1. ezEML user spaces**

ezEML currently creates a user space based on the identity provider's unique identifier in the authentication token. The same user may have multiple ezEML user spaces, one for each unique identifier. Mapping unique identifiers to a single user profile poses some risk to ezEML users if the separate user spaces are merged into a single user space based on the new user profile PASTA ID: the same EML data package information may occur in two or more user space locations for the same user and merging in the wrong order may affect the most accurate version of the data package information. Suggested options include:

1. Use the user profile PASTA ID to create a new user space and provide a mechanism to manually copy data package information from older user spaces into the PASTA ID user space.  
2. Retain the identity provider's unique identifiers in the authentication token based on the most recent authentication and sign in the ezEML user with that unique identifier, allowing access to an older user space.

**2. JSON Web Tokens**

1. The transport protocol in the HTTP Authorization header for JWTs will affect any non-EDI client using PASTA authentication tokens for replay requests since the authentication tokens are sent through the request's Cookie header. A suggested solution is to insert the JWT in both the Authorization and Cookies headers until a recommended deprecation period is met and then transition to only using the Authorization header.  
2. Embracing JWTs will require rewriting PASTA’s internal decoding of authentication tokens. To avoid a complete rewrite, only public-facing services (e.g., Data Portal, Gatekeeper, and ezEML) would initially need to support JWTs. The Gatekeeper could decode the JWT as the externally defined token and recast essential information into a PASTA authentication token for internal purposes.

# References

1. About Oauth2: https://oauth.net/2/  
2. About JSON web Tokens: https://jwt.io/introduction  
3. JSON Web Token standard RFC 7519: [https://datatracker.ietf.org/doc/html/rfc7519](https://datatracker.ietf.org/doc/html/rfc7519)  
4. About the EML metadata standard: https://eml.ecoinformatics.org/  
5. EML schema normative documentation: [https://eml.ecoinformatics.org/schema/](https://eml.ecoinformatics.org/schema/)  
6. Schema changes in EML 2.2.0: https://eml.ecoinformatics.org/whats-new-in-eml-2-2-0
