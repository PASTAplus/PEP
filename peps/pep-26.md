# PEP-26: Web-x as an EDI Service Documentation Application

- Author(s): Mark Servilla
- Contact: mark.servilla@gmail.com
- Status: Draft
- Type: Policy
- Created: 2026-08-24
- Reviewed:
- Final:

## Introduction

EDI has been prolific with the development of new public-facing services, including IAM, Drop, and now SAL. However, comprehensive user documentation describing how to interact with these services is either non-existent or limited behind locked repository walls. This PEP proposes the use of "web-x" as a dedicated documentation application to address this issue.

## Issue Statement

The development and deployment of EDI services has been rapid in recent years. Unfortunately, user documentation describing how to use these services has not kept pace. If documentation does exist, it is located on the service interface but is often not comprehensive or designed with a use-case scenario in mind, and its presentation format is typically inconsistent from service to service. (*ezEML* is an exception where extensive documentation has been integrated to the service application itself.) These are user issues that can easily be addressed by using *web-x* as an existing documentation solution.

 

## Proposed Solution

*Web-x* ([edirepository.org](https://edirepository.org)) is an in-house documentation service providing timely information about EDI to the user community. It is structured as a Markdown document repository (*content-x*), combined with a Markdown-to-HTML build capability that converts Markdown documents into a (mostly) static HTML website, *web-x*. Design intentions allow non-privileged users the ability to contribute Markdown documents to *content-x* via a "reviewed" GitHub PR without direct access to the *web-x* website. This workflow provides easy usability to both external and internal users of EDI and *web-x* meets the following criteria:

1. Centralized location for EDI service user documentation.
2. Consistent presentation of content.
3. Easy workflow using Markdown for presentation development.
4. Known website as domain *edirepository.org*.
5. User accessible without sign-in requirements.
6. Generates searchable content.

Additional requirements:

1. Content should only apply to user documentation. Architecture and API topics should be avoided unless relevant to user needs.
2. EDI service lead developer is responsible for content correctness and synchronization.

User documentation should be accessible as a subcategory under the *Support* menu with the title *User Documentation*. This link will refresh to a summary page with a list of all services having documentation, including a brief description of the service, where each service description will contain a primary link to the service documentation root page. Once at the documentation root page, page branching may occur as needed for use-case scenario organization. 

## Open issue(s)

## References

## Rejection
