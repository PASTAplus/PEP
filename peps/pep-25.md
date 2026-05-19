# PEP-25: System Activity Logger

## Overview

This document describes the design and implementation plan for a replacement for the PASTA Audit Manager service. The goal is to evolve the service from a narrow audit log for internal PASTA services into a unified, high-level event logger, capable of generating meaningful usage reports for both internal operators and end users.


## Motivation

The current Audit Manager has several limitations:

- XML report responses are buffered entirely in memory, causing risk with large result sets
- The `resource_reads` aggregation table is manually maintained in application code and can drift out of sync
- The schema contains significant redundancy (`userAgent`, `userid`) inflating storage on a high-volume append-only table
- The user-facing web UI is limited
- The schema conflates low-value fields (`category`, `statusCode`, `authSystem`, `groups`) with high-value ones, and lacks fields needed for future requirements (IP address, EDI token, geolocation)
- `serviceMethod` is a single opaque string; splitting into `service` + `method` improves queryability and normalization
- No mechanism for privacy-preserving user-facing reports


## Requirements

### Schema Requirements

| Field        | Notes                                                                      |
|--------------|----------------------------------------------------------------------------|
| `oid`        | Auto-generated row ID (primary key)                                        |
| `entryTime`  | Datetime timestamp, auto-generated on insert                               |
| `service`    | Name of the originating service (split from current `serviceMethod`)       |
| `method`     | Method name, e.g. `listRecentUploads` (split from current `serviceMethod`) |
| `entryText`  | Nullable; JSON field carrying method-specific context; not indexed         |
| `resourceId` | Nullable; identifies package, data object, metadata object, or report      |
| `userId`     | EDI user identifier — normalized into lookup table                         |
| `commonName` | Human-readable display name — normalized with `userId`                     |
| `userAgent`  | Full user agent string — normalized into lookup table                      |
| `referrer`   | Full refrerer string — notes on privacy below                              |
| `ipAddress`  | Client IP address — notes on privacy below                                 |
| `ediToken`   | JSON field; stored but never exposed or indexed                            |
| **Removed**  | `category`, `statusCode`, `authSystem`, `groups`                           |

### Functional Requirements

- All records must be queryable by: `service`, `method`, `resourceId`, `entryTime` range, `userId`
- Robot filtering must occur **before** records are submitted to the Audit Manager; the service itself does not filter robots
- User-facing reports must anonymize identity: real `userId` and `commonName` must never be exposed; substitute with `user-<hash>` or `user-N`
- IP addresses must never be exposed directly in user-facing reports; used only for internal geolocation enrichment
- EDI tokens must never be exposed in any API response

### Report Requirements

Users should be able to query:
- Number of unique (anonymized) users who downloaded a dataset
- Download counts per day / time series for a resource
- Geographic distribution of downloads (city-level geolocation from IP, subject to free-tier cost evaluation)
- Nice to have: Export of reports to PDF

### IP Address Field

**Data Collection**: The IP address field captures the client's IP address at the time an event is logged. This data can be used for various purposes, including geolocation mapping and identifying user locations or network details during specific events.

**Compliance with Privacy Regulations**: Ensuring compliance with privacy laws such as GDPR, CCPA, etc., requires that personal information (like IP addresses) be handled securely and not exposed publicly without adequate consent. The data should be treated confidentially for internal use only.

**Analytics Purpose**: Although primarily collected for detailed analytics within the EDI Repository system, the IP address is used to enrich geolocation details when generating user-facing reports or for other internal analytical purposes that do not compromise individual privacy.

### Implementation

**Anonymization**: Consider implementing a strategy to anonymize IP addresses immediately upon collection where possible. Techniques might include hashing algorithms (e.g., SHA-256) which transform the IP into a non-reversible format, or truncating parts of the IP address to lose its identifying value.

**Limited Use**: Ensure that IP addresses are used only for specific internal purposes such as geolocation enrichment in reports and not for any direct user identification or tracking beyond what is necessary for system analytics and maintenance.

**Transparency and Consent**: Where possible, inform users about the data collection practices including the use of their IP address, ensuring compliance with transparency requirements under privacy laws. This can be done through updated privacy policies that are clearly communicated to users before or upon using the service.

### Reporting Requirements

- Enrich geolocation details in reports where IP addresses have been anonymized, providing city and country information derived from the IP address but not exposing the raw IP itself.

### Referrer

The referrer header is used to indicate the URI of the previous web page from which a request was made.

**Logging Privacy**: In compliance with privacy requirements, the referrer field should not be exposed directly to users or in public-facing reports due to potential exposure of sensitive information about previous pages visited by users. It is recommended to normalize and anonymize this data as much as possible within the system.

**User Journey Analytics**: Although referrer data is collected for internal analytics, it should not be used for user identification or tracking beyond what is necessary for service improvement and usage monitoring. This could include understanding how users navigate through different services within EDI.

**Compliance with Regulations**: Ensure compliance with GDPR, CCPA, and other data protection regulations that may have specific requirements regarding referrer logging. The anonymization process should be robust enough to meet these regulatory standards.

#### Implementation

- **Anonymization**: Implement a hashing mechanism for the referrer field similar to how user identity is handled (e.g., `user-<hash>`) to protect privacy and ensure that raw referrer data is not stored or exposed in reports.

- **Internal Usage Only**: Ensure that all internal analytics tools and reports utilize the anonymized referrer data, avoiding any direct use of this information for external reporting purposes.

- **Privacy by Design**: Integrate referrer logging as part of a broader privacy strategy within the application architecture to minimize unnecessary data collection and ensure compliance with data protection laws.

### Reporting

Users or operators might be interested in:

- Anonymized counts of unique users who accessed specific resources or services, providing insights into popular paths through the system.
- Time series analysis for resource usage, possibly filtered by referrer to understand how different sources drive traffic.

### Future

Consider adding options for optional logging of referrer data under strict privacy guidelines and user consent mechanisms, allowing users to control the level of information shared about their browsing history.


## Design Decisions

### Schema Normalization

- Partially normalize the `eventlog` table to reduce storage on the two highest-volume repeated-value columns

- The main `eventlog` table references these by integer FK, reducing row size significantly on a high-volume append-only table. `authSystem`, `groups`, and `category` are dropped entirely.

### Replace `resource_reads` with a Materialized View

- The hand-maintained `resource_reads` table and its associated Java synchronization logic (`ReadsManager`, re-initialization script) are replaced by a PostgreSQL materialized view:

- Refreshed on a schedule (e.g. hourly via cron or pg_cron). This eliminates all application-level sync code and the re-initialization utility.

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

A separate query path for user-facing reports will:

- Never return raw `userId`, `commonName`, `ipAddress`, `referrer` or `ediToken`
- Replace user identity with a stable but non-reversible hash: `user-<SHA256(userId + salt)[0:12]>`
- Expose geolocation (city, country) derived from `ipAddress`, but not the IP itself
- Geolocation enrichment: evaluate MaxMind GeoLite2 (free tier) vs. paid alternatives as a separate spike

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

## Open Questions

- What is the acceptable staleness for the materialized view refresh interval?
- Should geolocation happen at ingest time or at query time?
- How to anonymize the more sensitive fields
- Should PDF report generation be server-side or client-side (browser print)?
