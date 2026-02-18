# PEP-22: PASTA Data Package Staging Service

Web service for staging data packages for upload to PASTA.

- Author(s): Mark Servilla, Roger Dahl
- Contact: 
- Status: Draft
- Type: Application
- Created: 2026-02-18
- Reviewed:
- Final:

## Introduction

This proposal describes the implementation of a service for staging data packages for upload to PASTA. 

## Issue Statement

Currently, users must have access to a 3rd party site that is accessible to PASTA over the internet in order to upload data packages through the API. This proposal aims to address this by providing a dedicated staging service with a public API and user interface for uploading and manging data before final upload to PASTA.

## Proposed Solution

### Development Approach

- Develop API first, then UI
- Use same stack as IAM service
  - Python with nginx + gunicorn + uvicorn + sqlalchemy + postgres
- Use web-x server for deployment testing
- Initial focus on regular uploads
  - Fault tolerance and network outage handling addressed later 

### API Requirements

- Publicly accessible API, protected by JWT bearer token or API key
- API key generated from IAM service
- API endpoints:
  - POST data with key and label
  - GET upload report/status
  - DELETE object
- POST returns 202 (Accepted) response for asynchronous processing

### User Interface

- Public-facing web UI for uploads
- Dashboard displaying all user uploads
- User-provided labels for tracking uploads to specific packages

### Authentication

- Integrate with IAM service for user authentication
- IAM will have separate set of resources for managing access to the staging service

### Backend Storage

- Use S3 bucket for staging storage (no filesystem shim)
  - This is a workaround for S3 shim 78GB limit
- Use boto3 library for S3 integration
- Configure Amazon ACLs:
  - Must be readable by PASTA
  - PASTA pulls data using existing pull mechanism
- Current option: Provide PASTA with functional URL for data retrieval
  - URLs may be private (not publicly accessible)
- Preferred approach: Rewrite PASTA to work with S3 directly

### Metadata and Database

- Store upload metadata in database
- Provide users access to upload information:
  - Upload status (success/failure)
  - Checksums (SHA-1/MD5 for EML validation)
  - Users can verify against local checksums
  - EML data link included in metadata

### Upload Status and Reporting

- Users can verify upload success
- Successful uploads provide retrieval URL
- Report page displays:
  - EML data links (functional URLs)
  - Time-to-live countdown for uploaded data
  - Cancel/delete option
- EML metadata not posted to staging server

### Data Lifecycle Management

- Garbage collection after configurable period (one month optimal for cost management)
- Amazon charges for access with one month granularity, aligning with GC strategy
- Timer starts at upload time

### Server Specifications

- Modest server requirements
- Checksums computed server-side

## Open issue(s)

## References

## Rejection
