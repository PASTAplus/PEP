# PEP-5: Transition from Flask to FastAPI

- Author(s): Roger Dahl
- Contact: dahl at unm edu
- Status: Draft
- Type: Application
- Created: 2024-08-20
- Reviewed:
- Final:

## Introduction

The EDI data repository uses a service oriented architecture, consisting of a number of RESTful (Representational State Transfer) web services. This PEP proposes standardizing on the FastAPI framework by moving the Python based services that are currently Flask based, over to FastAPI.

## Issue Statement

The EDI data repository currently uses Java and Python based web frameworks.

The Java based services use the Java Servlet API, and for some services, JAX-RS (Java API for RESTful Web Services), both which are part of the Java EE (Enterprise Edition) platform.

The Python based services use either the Flask or the FastAPI web framework.

Accounting for these variations, developers currently have to be familiar with four frameworks in order to maintain and develop new features for the data repository. 

## Proposed Solution

This PEP proposes moving the Python based services that are currently using the Flask web framework, over to the FastAPI framework.

This is anticipated to have the following benefits:

- Reducing the maintenance burden for EDI developers by removing one of the web frameworks which developers currently have to be familiar with in order to maintain and develop new features in the data repository.
- FastAPI is more modern and integrates tools from which we expect to benefit, such as OpenAPI for API documentation.
- FastAPI uses AsyncIO and therefore is expected to use fewer resources.
- As FastAPI uses fewer resources, it may reduce hosting costs at AWS.

## Open issue(s)

## References

## Rejection

