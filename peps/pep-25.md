# PEP-25: System Activity Logger

- Author(s): Roger Dahl
- Contact: dahl at unm edu
- Status: Draft
- Type: Application
- Created: 2026-05-19
- Reviewed:
- Final:


## Introduction

This document describes the design and implementation plan for a replacement for the PASTA Audit Manager service. The goal is to evolve the service from a narrow audit log for internal PASTA services into a unified, high-level event logger, capable of generating meaningful usage reports for both internal operators and end users.


## Issue Statement

The current Audit Manager has several limitations:

- XML report responses are buffered entirely in memory, causing risk with large result sets
- The `resource_reads` aggregation table is manually maintained in application code and can drift out of sync
- The schema contains significant redundancy (`userAgent`, duplicated identity fields) inflating storage on a high-volume append-only table
- The user-facing web UI is limited
- The schema mixes low-value fields (`category`, `statusCode`, `authSystem`, `groups`) with high-value ones, and lacks fields needed for future requirements (IP address, EDI token, geolocation)
- `serviceMethod` is a single opaque string; splitting into `service` + `method` improves queryability and normalization
- No mechanism for privacy-preserving user-facing reports


## Proposed Solution

### Schema Requirements

| Field       | Notes                                                                                                                                            |
|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| `id`        | Auto-generated row ID (primary key)                                                                                                              |
| `entryTime` | Datetime timestamp, auto-generated on insert                                                                                                     |
| `service`   | Name of the originating service (split from current `serviceMethod`)                                                                             |
| `method`    | Method name, e.g. `listRecentUploads` (split from current `serviceMethod`)                                                                       |
| `context`   | Nullable; `jsonb`;  carries method-specific context;                                                                                             |
| `resource`  | Nullable; identifies package, data object, metadata object, or report                                                                            |
| `userAgent` | Nullable; Full user agent string — normalized into lookup table                                                                                  |
| `referrer`  | Nullable; Full refrerer string — notes on privacy below                                                                                          |
| `ipAddress` | Nullable; `inet`; Client IP address — notes on privacy below                                                                                     |
| `ediToken`  | Nullable; `jsonb`; when present, authoritative token payload (includes `ediId`); never exposed; `ediToken->>'ediId'` is indexed for queryability |
| **Removed** | `category`, `statusCode`, `authSystem`, `groups`                                                                                                 |

### Functional Requirements

- All records must be queryable by: `service`, `method`, `resource`, `entryTime` range, and `ediToken.ediId` (`ediToken->>'ediId'`) where present
- The logger will assume that robot filtering has occurred before records are submitted to the logger; the service itself does not filter robots
- User-facing reports must anonymize identity: real identity (`ediToken->>'ediId'`) should not be exposed; substitute with `user-<hash>` or `user-N` and handle null token/identity as anonymous
- IP addresses must never be exposed directly in user-facing reports; used only for internal geolocation enrichment
- EDI tokens must never be exposed in any API response

### Report Requirements

Users should be able to query:
- Number of unique (anonymized) users who downloaded a dataset
- Download counts per day / time series for a resource
- Geographic distribution of downloads (city-level geolocation from IP, subject to free-tier cost evaluation)
- Nice to have: Export of reports to PDF

### Privacy-Sensitive Fields

`ipAddress`, `referrer`, and `ediToken` (including the identity it carries) are stored for internal use only and are **never returned in any API response or user-facing report** (see also [Functional Requirements](#functional-requirements)).

| Field                 | Stored purpose                          | Exposed in reports as                                                    |
|-----------------------|-----------------------------------------|--------------------------------------------------------------------------|
| `ipAddress`           | Geolocation enrichment (city, country)  | City and country only; raw IP omitted                                    |
| `referrer`            | Traffic-source and navigation analytics | Not exposed                                                              |
| `ediToken` / identity | User-count aggregation                  | Stable anonymous hash (`user-<SHA256(ediId+salt)[0:12]>`) or `anonymous` |

Compliance with GDPR, CCPA, and similar regulations is required; privacy policies must reflect this data collection.


## Design Decisions

### Schema Normalization

- Partially normalize the `eventlog` table to reduce storage on repeated-value columns while keeping identity data in the token payload

- The main `eventlog` table uses an integer FK for normalized `userAgent`. Identity is stored only in nullable `ediToken` (`jsonb`) rather than duplicated in a separate `ediId` column. Queryability by `ediId` is preserved with an expression index on `ediToken->>'ediId'`.

### Aggregated Counts

The current `resource_reads` table is maintained by application-level code, requiring multiple database round trips per log insert. The new schema's improved indexing may make these pre-aggregated counts unnecessary — aggregate queries will be evaluated against the main table once it is populated. If query performance proves insufficient, a replacement will be implemented using database triggers, eliminating the application-level sync logic entirely.

### Streaming Responses — XML and JSON

The current `GET /report` endpoint buffers the full result as a String in memory. Both XML and JSON report endpoints will be implemented using streams, consistent with the existing CSV endpoint pattern:

- `GET /report` → streaming `application/xml`
- `GET /jsonreport` → streaming `application/json`
- `GET /csvreport` → streaming `text/csv` (existing, retained)

### Split `serviceMethod` into `service` + `method`

Current single-string values like `DataPackageManager-1.0.readDataEntity` are split into:

- `service`: `DataPackageManager`
- `method`: `readDataEntity`

This improves index efficiency and enables independent filtering by service or method.

### Privacy-Preserving User Reports

A separate query path for user-facing reports will apply the privacy rules described in [Privacy-Sensitive Fields](#privacy-sensitive-fields):

- Replace user identity with a stable but non-reversible hash when identity is present: `user-<SHA256((ediToken->>'ediId') + salt)[0:12]>`; otherwise bucket as anonymous
- Expose geolocation (city, country) derived from `ipAddress`, but not the IP itself

### Geolocation Enrichment

IP-to-location resolution is performed asynchronously after ingest, decoupling it from the hot path of log record insertion. A background process will periodically resolve unprocessed `ipAddress` values and store the resulting city and country in a separate lookup table keyed by IP. Report queries join against this table to expose geolocation without ever surfacing the raw IP.

The geolocation data source will be evaluated as a separate spike; candidates include MaxMind GeoLite2 (free tier) and paid alternatives.

### Web UI

A user-facing web UI will be added following the same patterns as the DataPortal (Drop) UI:

- Allows users to query usage reports for a given dataset/resource
- Displays anonymized download counts, unique user counts, time series charts, geographic distribution
- PDF export of reports
- Operators get an additional non-anonymized view (authenticated, internal only)

### AuditCleaner

`AuditCleaner` is legacy code tied to a file-based XML caching approach that no longer exists. It will be **removed** in v2 with no replacement needed.

## Migration plan

- A one-time backfill script will be run to populate the new schema with historical data from the old table.
- During migration, legacy `ediId` values will be copied into `ediToken` and the standalone `ediId` column will be removed.
- Add an expression index on `ediToken->>'ediId'` (ideally a partial index where `ediToken` is not null) before switching report queries to the new schema.

## Open issue(s)

- Should PDF report generation be server-side or client-side (browser print)?
