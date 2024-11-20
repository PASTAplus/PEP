# PEP-3: Transition proprietary PASTA token format to JSON Web Tokens (JWT)

- Author(s): Roger Dahl
- Contact: dahl at unm edu
- Status: Draft
- Type: Application
- Created: 2024-06-18
- Reviewed: 2024-08-28
- Final:


## Introduction

In the process of a user executing operations within PASTA, a bearer token, referred to as a PASTA token below, is passed to the various microservices that constitute the PASTA infrastructure. This PEP proposes transitioning these PASTA tokens from a proprietary format to standard JSON Web Tokens (JWT).


## Issue Statement

PASTA uses tokens to carry claims about the user to the various microservices that make up the system. By passing the claims directly, we avoid having to query the DB for this information in each microservice, as would be the case if we were using opaque bearer tokens.

When PASTA was initially developed, token representations had yet to be standardized, and a proprietary format was developed for PASTA. While this format has served us well, it has the following issues:  

- As we move forward with planned new features related to managing users and access control in PASTA, we anticipate requiring the PASTA token to hold additional claims. The current proprietary format is not well suited to be extended to hold these new claims. It would also require us to update implementations with custom code to handle token serialization and deserialization.

- It is unfamiliar and hard to work with for new maintainers.


## Proposed Solution

Several industry standards for representing bearer tokens are now available, one of which is the JSON Web Token (JWT). We will move to this standard, which has the following benefits:

- We can simplify source code maintenance by removing the custom code to serialize and deserialize the propriety PASTA tokens, and replace it with widely used standard libraries. We know that these libraries are battle tested and secure.

- JWTs hold nested data and can be easily extended without having to modify serialization and deserialization code.

- JSON is familiar to most software developers, and is easy to work with. By extension, JWTs are as well.

The transition will be performed in multiple steps, as follows:

- The proprietary token will be left unmodified, and will remain available for use with legacy code.

- The new JWT token will be made available as an option to use instead of the proprietary token.

- Over time, microservices are transitioned to use the new token.

- Any additional information that is required in the token as we move forward with new features related to managing users and access control in PASTA, will only be added to the new JWT token. To use the new claims or other information, services must transition to the new token.

- When there are no remaining consumers of the proprietary token, we stop including it in the auth microservice responses and remove the associated code.

### JWT Token Structure

A JWT token consists of three parts separated by dots: the header, the payload, and the signature. The header and payload are JSON objects, and the signature is a hash of the header, payload, and a secret key. The header and payload are base64 encoded, and the signature is base64 encoded and hashed.

#### JWT payload for token returned by EDI Authn

The following claims are included in the payload of JWT tokens returned by the EDI Authentication (Authn) service:

| Claim                | Description                     |
|----------------------|---------------------------------|
| sub                  | PASTA user profile identifier   |
| cn                   | Common name                     |
| gn                   | Given name                      |
| email                | Email address                   |
| hd                   | Hosted domain                   |
| iss                  | Issuer                          |
| sn                   | System notification             |
| iat                  | Issued at                       |
| nbf                  | Not before                      |
| exp                  | Expiration                      |
| pastaGroups          | List of PASTA group identifiers |
| pastaIsEmailEnabled  | Email notifications enabled     |
| pastaIsEmailVerified | Email address verified          |
| pastaIsVetted        | User is vetted                  |
| pastaIsAuthenticated | User is authenticated           |
| pastaIdentityId      | PASTA Identity ID               |

Specifics on usage of these claims in the context of PASTA:

- `sub`: Unique, immutable identifier for a user profile in PASTA. This identifies the user for which the token was issued. It is created when a user creates a profile. The identifier is on the form `PASTA-<UUID-HEX>`, where `<UUID-HEX>` is a 32-character hexadecimal string. We refer to identifiers on this form as PASTA identifiers. There are two types of PASTA identifiers; one for users and one for groups. 
- `cn`: Full name of the user. This is a free form field which may be modified by the user. It is meant for display only and no semantics should be inferred.
- `gn`: First name of the user. This contains the first word of the `cn`, or the full `cn` if the `cn` contains only one word.
- `email`: The user's email address. This is the email address provided by the user.

Token metadata:

- `hd`: The hosted domain. Always set to `edirepository.org`.
- `iss`: The issuer. Always set to `https://edirepository.org`.
- `sn`: System notification flag, currently always set to `false`. This may be used in the future to indicate that there is a system notification for the user.

Timestamps:

- `iat`: Unix timestamp indicating when the token was issued.
- `nbf`: Unix timestamp indicating when the token becomes valid. This will always match the `iat` claim.
- `exp`: Unix timestamp indicating when the token expires. Tokens currently have a lifetime of 8 hours. Tokens that have not expired are automatically refreshed when using the Authn UI, and can also be refreshed programmatically with an API call.

Custom claims:

- `pastaGroups`: List of PASTA group identifiers, with cardinality zero to many. This lists groups in which the user is a member. A group grants the user access to the resources to which the group has access. A group may be owned by the user, in which case they will have added themselves to the group, or by another user, in which case they will have been added by the other user. Users have full control over memberships in groups owned by themselves, but can only leave, not join, groups owned by others. 
- `pastaIsEmailEnabled`: Flag that the user has approved sending automated notifications to the provided email address. Even if this flag is not set, we may send important emails to the user.
- `pastaIsEmailVerified`: Flag that we have verified that the user is able to receive emails at the address provided in `email`.
- `pastaIsVetted`: Flag indicating whether the user has been vetted by EDI. Vetted users are granted elevated access in PASTA.
- `pastaIsAuthenticated`: Flag indicating that the user has signed in. This will be `true` if the user has authenticated via EDI LDAP or via one of the supported 3rd party identity providers. If will be `false` if the token has been automatically issued to a public, unauthenticated user.
- `pastaIdentityId`: Internal PASTA Identity ID. This is a unique identifier for the identity provider and account used for signing in to the profile. Since a user may have multiple accounts linked to their profile, this identifier can change with each sign-in. It is used internally in the Authn account linking procedure and may be used in future APIs to provide information about the user's identity provider and current accounts. If the token is issued to a public, unauthenticated user, this claim will have value `0`.


#### Example payload

```json
{ 
  "sub": "PASTA-b4022bba0474451785dc244143c29f4d",
  "cn": "Mary Smith",
  "gn": "Mary",
  "email": "mary@marysmith.org",
  "hd": "edirepository.org",
  "iss": "https://edirepository.org",
  "sn": false,
  "iat": "1731516687",
  "nbf": "1731516687",
  "exp": "1731520287",
  "pastaGroups": [
    "PASTA-3e592810230b45e3933a8341da02a873",
    "PASTA-80ffbe75703f46d8b2fe3b2c7af67b5f",
    "PASTA-a597bcf3f3414fe783c230d4e9c6e4f6"
  ],
  "pastaIsEmailEnabled": true,
  "pastaIsEmailVerified": true,
  "pastaIsVetted": false,
  "pastaIsAuthenticated": true,
  "pastaIdentityId": 49322
}
```

## Open issue(s)


## References
