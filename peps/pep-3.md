# PEP-3: Transition proprietary PASTA token format to JSON Web Tokens (JWT)

- Author(s): Roger Dahl
- Contact: dahl at unm edu
- Status: Draft
- Type: Application
- Created: 2024-06-18
- Reviewed:
- Final:


## Introduction

In the process of a user executing operations within PASTA, a bearer token, referred to as a PASTA token below, is passed to the various microservices that constitute the PASTA infrastructure. This PEP proposes transitioning these PASTA tokens from a proprietary format to standard JSON Web Tokens (JWT).


## Issue Statement

PASTA uses tokens to carry claims about the user to the various microservices that make up the system. By passing the claims directly, we avoid having to query the DB for this information in each microservice, as would be the case if we were using opaque bearer tokens.

When PASTA was initially developed, token representations had yet to be standardized, and a proprietary format was developed for PASTA. While this format has served us well, it has the following issue:  

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


## Open issue(s)


## References
