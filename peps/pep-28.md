# PEP-28: Model Context Protocol (MCP) Server for the PASTA+ REST API

- Author(s): Colin Smith
- Contact: colin.smith@wisc.edu
- Status: Draft
- Type: Application
- Created: 2026-10-13
- Reviewed:
- Final:

## Introduction

The Model Context Protocol (MCP), developed by Anthropic and rapidly adopted across the artificial intelligence ecosystem, is an open standard that allows large language models (LLMs) and autonomous AI agents (such as Claude Desktop, ChatGPT, Cursor, and custom Python-based agentic workflows) to discover and interact securely with external tools, contextual resources, and specialized prompt templates. 

The Environmental Data Initiative (EDI) operates the PASTA+ data repository, providing open, curated scientific and ecological data packages via a web-based data portal and a comprehensive REST API. While users traditionally discover and retrieve data through manual portal exploration or scripted API workflows, modern scientific workflows are rapidly shifting toward autonomous agentic synthesis, analysis, and intelligent metadata curation.

This proposal describes the design, deployment, governance, and security model for an official **PASTA+ MCP Server**. By exposing PASTA+ through the MCP standard, EDI will enable AI agents to natively query, inspect, and retrieve scientific data while maintaining strict safeguards against denial-of-service (DoS) attacks, agent runaways, unbuffered memory exhaustion, and unauthorized repository mutations. Furthermore, this document outlines a sustainable, tiered service model that accommodates users across different operational scales—from individual researchers conducting interactive exploratory queries to intensive project workflows running high-frequency automated synthesis—ensuring fair resource allocation while recovering compute and bandwidth costs proportionate to utilization.


## Issue Statement

### The Opportunity: Agentic Data Discovery and Synthesis
Scientific data repositories face an accessibility paradox: while millions of ecological observations, sensor streams, and taxonomy datasets are stored across tens of thousands of data packages in PASTA+, finding, synthesizing, and interpreting these datasets remains labor-intensive.

By far the most common mode of interacting with PASTA+ is through manual navigation of the web-based EDI Data Portal. In this conventional workflow, researchers manually formulate search queries, sift through extensive lists of results, inspect individual dataset landing pages, read through complex Ecological Metadata Language (EML) documents, and download tabular data files one by one. This manual discovery and evaluation process is exceptionally labor-intensive, repetitive, and presents a severe bottleneck when researchers attempt cross-site ecological syntheses spanning dozens or hundreds of packages.

A smaller subset of computationally proficient users interacts with PASTA+ programmatically by authoring one-off scripts to query the Solr search index and parse EML XML payloads directly. While programmatic, these custom scripts represent a minor fraction of overall user interactions, require specialized programming effort, and remain brittle in the face of varying data package content.

Enabling AI agents to directly interact with PASTA+ via standardized MCP tools and resources bridges both interaction modes: it reduces the heavy manual labor of portal-based searching and document inspection for the vast majority of researchers, while replacing brittle, one-off scripts with standardized, conversational agentic discovery, automated synthesis, and metadata curation.

### The Operational Challenge: Bot Traffic, Scrapers, and Repository Strain
EDI has experienced severe operational challenges stemming from aggressive robotic scraping and distributed denial-of-service (DDoS) incidents. These bot incursions have repeatedly degraded repository performance and forced EDI to devote substantial time to refactoring access control systems (e.g., establishing mandatory authentication for repository access, introducing user-managed API keys, and defining verified human authorization tiers).

Deploying an AI agent interface without strict architectural boundaries introduces operational risks that must be mitigated:

1. **Denial-of-Service from Agentic Feedback Loops**: Autonomous agents equipped with tool-calling capabilities often enter iterative reasoning loops. An unconstrained agent can trigger dozens of resource-intensive Solr queries or multi-gigabyte data entity downloads within seconds, overwhelming the PASTA+ server and egress pipelines.
2. **"Agent Breakout" and Repository Mutation Concerns**: Allowing AI agents to invoke mutating operations (such as data package creation/update) creates the risk of unintended repository modifications, corrupted package revisions, or upload queue congestion caused by hallucinated or unvalidated agent actions.
3. **Memory Buffering and Resource Exhaustion**: Buffering large XML payloads or data entities entirely in application memory creates severe server instability and out-of-memory risks. Any intermediate proxy or server must avoid in-memory buffering when serving multi-megabyte or multi-gigabyte datasets, relying instead on non-blocking asynchronous streaming.
4. **Supporting Diverse User Workloads Sustainably**: User workflows interact with the repository at vastly different scales. A casual researcher asking an assistant to locate a few relevant datasets generates minimal load. Conversely, an automated research lab pipeline running cross-site synthesis across hundreds of packages or performing batch metadata congruence evaluations generates intense, sustained traffic. Providing unbounded access to high-throughput agent loops risks degrading service for the broader community, while flat rate-limiting would artificially stifle productive, large-scale scientific applications. A tiered access framework allows users to scale their agentic interactions to match their workflow demands while contributing proportionally to the repository's cloud infrastructure expenses.


## Proposed Solution

To resolve these challenges, we propose an **EDI-Hosted Cloud Gateway Architecture** for the PASTA+ MCP Server built with Python and FastAPI, governed by multi-layer bot mitigation and rate-limiting infrastructure on AWS, backed by a phased safety rollout, and supported through structured operational capacity tiers.

In this architecture, client agents (such as Claude Desktop, Cursor, or custom research workflows) connect over HTTPS to an EDI-hosted cloud gateway using Server-Sent Events (SSE) or Streamable HTTP. Incoming requests pass through edge perimeter defenses—including bot filtering, API key verification, and rate limiting—before reaching the PASTA+ MCP Server application. The MCP server executes tool requests, streams data asynchronously, and communicates with PASTA+ backend services through internal proxy routing and intermediate caching layers.

### 1. EDI-Hosted Cloud Gateway Architecture (Remote SSE / Streamable HTTP)

The MCP server is engineered as a secure, EDI-managed service deployed exclusively within EDI's AWS cloud infrastructure using FastAPI and the official Python MCP SDK (`mcp` / `fastmcp`), utilizing remote Server-Sent Events (SSE) and Streamable HTTP transports:

1. **Centralized Cloud Gateway Deployment**:
   - Exposes standard Model Context Protocol endpoints over Server-Sent Events (SSE) and Streamable HTTP (e.g., `https://mcp.edirepository.org/sse`).
   - Fronted by AWS perimeter shielding (such as AWS WAF and an edge API gateway) to establish a unified perimeter providing mandatory API key authentication, tier quota enforcement, automated bot defense, response caching, and metered usage.
   - Directly connects desktop AI assistants (e.g., Claude Desktop, Cursor), cloud agent environments, and research synthesis pipelines over secure HTTPS without requiring end users to install local runtimes, manage Python environments, or configure backend dependencies.
2. **Security & Zero Client Code Distribution**:
   - In adherence to EDI repository security policies that restrict public disclosure of repository codebase logic, EDI does not publish server or client packages to public package indexes (such as PyPI) or public source repositories.
   - Hosting the MCP server strictly within EDI's managed AWS perimeter prevents disclosure of internal service implementations and guarantees that all agent interactions pass through perimeter controls and token-bucket throttles, eliminating any possibility of client-side bypass of rate limits or access controls.
3. **Client Integration & Transport Compatibility**:
   - Modern MCP-compliant desktop tools and agent frameworks connect natively to the remote SSE endpoint using HTTPS and the user's EDI API key configured in request headers (`X-API-Key: <key>`).
   - For desktop clients or legacy environments that strictly require local standard input/output (`stdio`) process execution, users can connect through standard, open-source third-party transport bridges (such as the generic `mcp-proxy` or `@modelcontextprotocol/server-sse` runner) that tunnel local `stdio` to the EDI remote SSE gateway over HTTPS, requiring no EDI-specific software distribution.


### 2. Bot Mitigation, Traffic Governance, and DoS Protection

To protect the PASTA+ core cluster from automated agent surges and minimize operational disruption, the hosted MCP service employs a multi-tiered defense:

1. **Mandatory EDI API Key Authentication**:
   - In adherence to EDI repository access policies and IAM security controls, anonymous requests are strictly rejected. Every MCP session must present an active EDI API key via the `X-API-Key` header or MCP connection configuration.
   - API keys are verified at the cloud perimeter gateway layer before any request reaches the MCP application or PASTA+ backend.
2. **Edge Throttling and Rate Limiting**:
   - The perimeter gateway enforces per-key rate limits and burst quotas. If an agent loops excessively or enters a runaway execution state, requests exceeding configured thresholds are immediately shed with HTTP 429 ("Too Many Requests") responses at the edge, ensuring 0% surplus load reaches the PASTA+ backend.
3. **AWS WAF Bot Mitigation & IP Reputation**:
   - AWS WAF operates at the perimeter, applying managed bot control rule sets, blocking known malicious scanning networks, and inspecting request frequency to thwart distributed volumetric scraping attempts.
4. **Agent Traffic Segregation and Header Tagging**:
   - When the MCP server proxies requests to the EDI Gatekeeper, it automatically injects:
     - `User-Agent: EDI-MCP-Agent/1.0 (AWS-Hosted; +https://edirepository.org)`
     - `EDI-Agent-Client-ID: <hashed-user-id>`
   - This header tagging allows the Gatekeeper and the PASTA+ System Activity Logger to immediately distinguish legitimate, authenticated AI agent traffic from illicit scrapers and anonymous web crawlers.
5. **High-Performance Intermediate Caching**:
   - High-frequency, expensive read operations—including Solr searches (`searchDataPackages`), EML metadata documents (`readMetadata`), Dublin Core XML, and evaluation reports—are cached in an intermediate in-memory store (such as Redis / Amazon ElastiCache) with configurable Time-To-Live (TTL, e.g., 1 hour for searches, 24 hours for immutable package revisions).
   - Identical agent queries are served directly out of the cache without dispatching HTTP requests to backend PASTA+ servlets.


### 3. Phased Capability Scoping & Sandboxing

To address concerns regarding "agent breakouts" and accidental repository corruption, the MCP server will implement a strict phased capability rollout:

| Phase | Operational Scope | Permitted Environments | Guardrails & Safeguards |
| :--- | :--- | :--- | :--- |
| **Phase 1 (Launch)** | **Strictly Read-Only** | Production (`pasta.lternet.edu`) & Staging (`pasta-s.lternet.edu`) | Zero mutation tools exposed. Agents can only search, discover, inspect metadata, read evaluation reports, and download data entities. |
| **Phase 2 (Future)** | **Mutation & Upload** | **Staging Environment Only** (`pasta-s.lternet.edu`) | Data evaluation (`evaluateDataPackage`) and upload (`createDataPackage`, `updateDataPackage`) strictly gated behind Human-in-the-Loop (HITL) approval tokens. Cannot target Production. |

In Phase 1, there is zero risk of an agent altering repository state, overwriting identifiers, or consuming upload processing queues. In Phase 2, agents will be restricted to staging sandboxes and must receive interactive user confirmation before submitting evaluate or upload transactions.


### 4. MCP Primitives: Tools, Resources, and Prompts

The PASTA+ MCP server will implement the three core primitives of the Model Context Protocol:

#### 4.1 Tools (Actionable Functions)

##### Phase 1: Read-Only Tools (Production & Staging)

###### Discovery & Search
- `searchDataPackages`
- `listDataPackageScopes`
- `listDataPackageIdentifiers`
- `listDataPackageRevisions`
- `listRecentChanges`

###### Metadata & Quality Inspection
- `readMetadata`
- `readDataPackageReport`
- `readDataEntityNames`
- `readDataEntitySize`
- `readDataEntityChecksum`

###### Data Access & Downloads
- `readDataEntity`
- `readDataPackageArchive`

###### Citations & Provenance
- `getCitation`
- `listDataPackageCitations`
- `getProvenanceMetadata`

##### Phase 2: Mutation Tools (Staging Sandbox Only)
- `evaluateDataPackage`
- `createDataPackage`
- `updateDataPackage`

#### 4.2 Resources (URI-Addressable Context)

Resources provide static or dynamic read-only context that can be attached directly into the agent's prompt context:
- `pasta://package/{scope}/{identifier}/{revision}/eml`: Raw XML metadata document for a package.
- `pasta://package/{scope}/{identifier}/{revision}/quality-report`: Complete XML/text quality report.
- `pasta://package/{scope}/{identifier}/{revision}/citation`: Canonical citation and DOI.
- `pasta://schema/eml-2.2.0`: Standard EML 2.2.0 XML schema definitions for agent-driven schema validation.

#### 4.3 Prompts (Guided Domain Workflows)

Standardized prompt templates embedded within the MCP server guide LLMs through complex ecological workflows:
- `dataset_discovery_assistant`: Prompts the agent to interview the user regarding ecosystem type, spatial boundaries, temporal ranges, and taxa, generating structured Solr queries for PASTA+.
- `eml_metadata_auditor`: Instructs the agent to inspect a package's EML and quality report, identifying missing attribute definitions, non-standard unit declarations, or license compliance gaps.
- `cross_package_synthesis`: Guides the agent in joining tabular data entities across multiple LTER or EDI packages with heterogeneous column schemas.


### 5. Data Entity Handling: Out-of-Band Retrieval and Context Boundaries

PASTA+ data entities range from small tabular text files to multi-gigabyte CSVs, raster grids (GeoTIFF), NetCDF cubes, or binary archives. In agentic workflows, large data entities are **not** ingested as raw tokens into the LLM context window, as doing so would rapidly exhaust token limits, degrade reasoning quality, or corrupt the context with binary data.

Instead, the MCP server supports a clear architectural separation between **in-context reasoning** and **out-of-band data processing**:

1. **In-Context Metadata & Schema Inspection**:
   - Consuming agents use MCP tools (`readMetadata`, `readDataEntityNames`, `readDataEntitySize`, `readDataEntityChecksum`) to inspect column definitions, units, schema, and byte sizes directly within their prompt context window.
   - This compact structural metadata (typically a few hundred to thousand tokens) enables the agent to understand dataset schemas, formulate analytical plans, and write accurate data processing code.

2. **Out-of-Band Data Consumption via Code Execution**:
   - For actual analysis or synthesis, the agent generates executable code targeting the canonical PASTA+ data entity URL (`https://pasta.lternet.edu/package/data/eml/{scope}/{identifier}/{revision}/{entityId}`).
   - The code executes in a local environment or sandbox, downloads the file directly to disk or memory, performs required filtering, aggregation, or joining, and returns only concise summaries, statistical tables, or preview slices back to the prompt context.

3. **Non-Buffering Streaming for Direct Downloads**:
   - When client applications, automated pipelines, or user scripts invoke `readDataEntity` or `readDataPackageArchive` through the MCP gateway, the FastAPI server acts as a non-buffering passthrough.
   - Using asynchronous chunked transfer streaming, payloads pass straight from PASTA+ to client storage without accumulating in memory buffers on either the MCP gateway or PASTA+ nodes.
   - The server forwards standard HTTP headers (`Content-Length`, `Content-Type`, and `Content-Disposition`) so client environments can inspect payload size and MIME type before initiating file transfers.


### 6. Sustainable Operational Model & Capacity Tiers

Operating high-throughput AI agent gateways incurs cloud infrastructure overhead (compute runtime, network data egress, and intermediate cache memory). To ensure long-term financial sustainability and equitable resource allocation across diverse scientific use cases, the hosted MCP service will offer structured capacity tiers based on workload scale.

#### 6.1 Architecture for Tier Governance

To enforce usage quotas cleanly without fragmenting user accounts or locking into rigid cloud-provider mechanisms:

1. **Centralized Identity and Tier Authority (EDI IAM)**:
   - Capacity tiers are represented natively within the EDI Identity and Access Management (IAM) directory (e.g., IAM groups `mcp-starter`, `mcp-team`, `mcp-power`).
   - Maintaining tier definitions within EDI IAM preserves a single source of truth, integrates naturally with existing EDI user accounts and ORCID credentials, and avoids complex state synchronization between EDI and external cloud services.
2. **Layered Defense & Quota Enforcement**:
   - **Perimeter Layer (WAF & Edge Gateway)**: Evaluates incoming requests at the cloud ingress, handling coarse volumetric bot filtering, IP reputation checks, and initial request validation before requests consume internal resources.
   - **MCP Application & Cache Layer**: Enforces per-tier quotas, rate limits, and tool permissions based on the user's authenticated IAM entitlements, leveraging the intermediate cache store for fast, stateful quota tracking.

#### 6.2 Tier Structure

| Feature / Tier | Starter / Exploratory (Free) | Researcher / Team | Power / High-Volume |
| :--- | :--- | :--- | :--- |
| **Target Audience** | Individual researchers, students, exploratory desktop agents | Research working groups, lab pipelines, synthesis projects | High-throughput synthesis engines, large-scale automation, institutional aggregators |
| **IAM Group** | `mcp-starter` | `mcp-team` | `mcp-power` |
| **Billing Model** | Free (Requires verified EDI user account) | $25 – $75 / month (or project grant fee) | $500 – $1,000+ / month base + metered data egress |
| **Rate Limit (Burst / Sustained)** | 10 req/sec burst; 60 req/min sustained | 50 req/sec burst; 300 req/min sustained | 200 req/sec burst; 1,200 req/min sustained |
| **Daily Quota** | 1,000 calls / day | 25,000 calls / day | 250,000+ calls / day (customizable) |
| **Caching Layer** | Aggressive Redis caching (1h – 24h) | Standard caching with bypass headers | Dedicated cache allocation / real-time read |
| **Phase 2 Staging Access** | Read-only | Included (Staging evaluate/upload sandbox) | Included (High-concurrency staging pipelines) |
| **S3 Egress Optimization** | Proxied through PASTA+ web servlet | Proxied through PASTA+ web servlet | Direct Presigned S3 URLs (PASTA+ S3 Object Storage) |
| **SLA & Support** | Best-effort community support | 99.5% uptime SLA, email support | 99.9% uptime SLA, dedicated support channel |
| **Billing Gateway** | N/A | Stripe Billing / EDI Portal recharge | Invoiced / Cloud Marketplace Contract |

*Note: The rate limits, quotas, and pricing targets listed in this table represent illustrative baseline tiers. Operational thresholds, burst allowances, and quota policies will be calibrated during deployment.*

#### 6.3 Cost Recovery and Sustainability Impact
By aligning tiers with operational scale rather than institutional identity, EDI ensures that individual researchers and students retain free, frictionless access for interactive exploration. Simultaneously, users operating high-throughput automated synthesis pipelines or institutional batch workflows contribute proportionately to infrastructure costs, turning potential repository strain into a sustainable funding mechanism for EDI's cloud footprint.


### 7. Effects on Related EDI Systems

- **Gatekeeper (Reverse Proxy)**: 
  - Offloaded from handling high-frequency repeated searches due to MCP Redis caching.
  - Receives pre-authenticated requests bearing valid EDI API keys and explicit `EDI-MCP-Agent/1.0` headers, eliminating false-positive bot-blocking triggers.
- **Identity and Access Management (IAM)**:
  - Validates API keys presented by the MCP server through the centralized IAM authentication service.
  - Acts as the central authority for capacity tier management via IAM groups (`mcp-starter`, `mcp-team`, `mcp-power`). The returned EDI authentication token encodes group memberships, allowing the MCP gateway to enforce tier quotas without additional database queries.
- **Data Package Manager**:
  - Shielded from volumetric query storms by perimeter and gateway rate limiters.
  - Receives well-formed, standard REST calls without needing modification.
- **System Activity Logger (Audit Manager)**:
  - The distinct `EDI-MCP-Agent/1.0` user-agent and client ID enable clean segregation of AI agent metrics from human portal downloads, preventing statistical skew in dataset usage reports.
- **AWS S3 Object Storage**:
  - In Power / High-Volume tiers, large data entity downloads can be served via presigned S3 URLs generated directly from EDI's cloud object storage, offloading heavy multi-gigabyte transfers completely from the PASTA+ application servers.


### 8. Cloud Infrastructure & Cost Model

The hosted MCP service leverages standard cloud components within EDI's managed AWS environment:

| Cloud Component | Architectural Role | Operational Cost Driver |
| :--- | :--- | :--- |
| **Edge Gateway & WAF** | Perimeter routing, TLS termination, bot filtering, and DDoS defense | Request volume and inspection rule sets |
| **Compute Service** | Hosting the containerized FastAPI MCP application | Compute runtime hours and memory footprint |
| **Intermediate Cache** | In-memory caching for metadata, query results, and quota state | Cache memory capacity and persistence needs |
| **Data Egress** | Delivering package payloads and data streams to client agents | Outbound network bandwidth (offset by higher-tier allocations) |

Specific instance types, provisioning options, and autoscaling thresholds will be defined during deployment planning. Baseline operational expenses for the core service are modest, and higher-capacity tier subscriptions are structured to recover incremental compute and network egress overhead sustainably.


### 9. Alternative Solutions Considered

Before selecting the Model Context Protocol (MCP), three alternative architectural patterns for enabling AI agent access to the PASTA+ REST API were evaluated:

#### 9.1 Direct OpenAPI Specification Ingestion (Declarative Function Calling)
- **Description**: Publish a comprehensive OpenAPI 3.0 / 3.1 specification (JSON/YAML) for the PASTA+ REST API, allowing agent orchestration frameworks (OpenAI GPT Actions, LangChain `OpenAPIToolkit`, Anthropic tool use) to parse endpoint schemas dynamically at runtime.
- **Why Not Recommended**:
  1. *Excessive Token Overhead*: PASTA+ exposes dozens of endpoints with extensive parameters and XML schema requirements. Injecting complete OpenAPI schemas into an agent's context consumes thousands of tokens per reasoning step, increasing prompt costs and saturating context windows.
  2. *XML Processing Barrier*: LLMs struggle with generating and validating raw XML documents compared to JSON.
  3. *Lacks Desktop Client Support*: OpenAPI specs cannot be plugged directly into widely used desktop AI clients (Claude Desktop, Cursor, Zed) without writing custom intermediate bridge code.
  4. *Absence of Resources and Prompts*: OpenAPI only defines callable endpoints; it provides no native mechanism for exposing URI-addressable static/dynamic reference resources (`pasta://...`) or pre-built curation prompts.

#### 9.2 Agent-Optimized Python Client Library ("Code-as-a-Tool")
- **Description**: Develop and distribute an idiomatic Python library specifically designed for AI agents operating in code-execution sandboxes (e.g., Jupyter notebooks, Python REPL tools). The library would wrap PASTA+ endpoints, parse XML responses into pandas DataFrames and dictionaries, and execute data processing locally in the client's execution environment.
- **Why Not Recommended as the Primary Solution**:
  1. *Requires Code Execution Runtime*: This approach only functions for agents equipped with an active Python interpreter or sandboxed shell; it cannot be used by conversational or GUI-based agents lacking code execution capabilities.
  2. *Inaccessible to Desktop Agents*: Desktop assistants like Claude Desktop cannot consume Python SDKs directly.
  3. *Client Maintenance Burden*: Updates to API behaviors or schema handling require client-side library upgrades across user environments.
  - *Note*: While not recommended as the primary interface, an agent-optimized Python SDK could serve as a valuable complementary tool in future development for Jupyter-based analytical workflows.

#### 9.3 LLM-Tailored REST/JSON Middleware Proxy
- **Description**: Deploy a custom REST proxy that translates PASTA+ XML endpoints into lean JSON responses and exposes an endpoint (e.g., `GET /tools/schema`) returning OpenAI/Anthropic function definitions.
- **Why Not Recommended**:
  1. *Non-Standard Integration*: Lacks an industry-standard client specification. Every client framework would require custom configuration to consume the custom REST proxy.
  2. *Equivalent Hosting Overhead without Ecosystem Benefits*: Requires hosting, monitoring, and maintaining an intermediate web service similar to an MCP server, but forfeits native compatibility with the rapidly expanding MCP ecosystem.

#### Summary Comparison of Alternatives

| Criterion | Model Context Protocol (MCP) | Direct OpenAPI Ingestion | Agent-Optimized Python SDK | LLM-Tailored REST Proxy |
| :--- | :--- | :--- | :--- | :--- |
| **Protocol / Transport** | JSON-RPC 2.0 (Remote SSE / HTTP) | Stateless HTTP REST | Python method calls | Stateless HTTP REST |
| **EDI Hosting Footprint** | FastAPI container in AWS | Static file hosting | PyPI package maintenance | FastAPI container |
| **Client Code Distribution** | **None (Zero code distributed)** | OpenAPI spec file only | Public PyPI package distribution | None |
| **Desktop Client Support** (Claude Desktop, Cursor) | **Native out-of-the-box** (Remote SSE) | Requires custom bridges | No (requires code sandbox) | Requires custom bridges |
| **Context Token Efficiency** | High (Structured tool schemas) | Low (Monolithic schemas in prompt) | **Very High** (Local in-memory filtering) | High (Compact JSON) |
| **Payload Format Handling** | Handled by server tool wrappers | Raw PASTA+ XML | Parsed to DataFrames/Dicts | Translated to JSON |
| **Support for Resources & Prompts**| **Yes** (Native primitives) | No (Tools only) | No (Tools only) | No (Tools only) |
| **Traffic & Rate Limit Defense** | Enforced at MCP/WAF layer | Gatekeeper / WAF only | Uncontrolled client calls | Enforced at proxy layer |


## Open issue(s)

1. **Cloud Compute Runtime & Deployment Topology**:
   The specific compute hosting environment and deployment topology for the FastAPI MCP application (e.g., containerized service vs. serverless execution) will be determined during implementation based on EDI's VPC layout, streaming requirements, and operational monitoring standards.
2. **Human-in-the-Loop (HITL) Token Specification for Phase 2**:
   For Phase 2 evaluation and upload tools, an exact cryptographic token mechanism (e.g., short-lived signed JWTs generated via the EDI Data Portal after human review) must be designed to verify that a human operator explicitly authorized the staging upload.
3. **Billing Integration Platform**:
   Whether capacity tier subscriptions should be managed via an integrated payment processor on the EDI website/portal, institutional recharge, or cloud marketplace listing will be evaluated prior to offering paid tiers.
4. **Agent Data Ingestion Patterns and Client Guidelines**:
   Because PASTA+ data entities can be multi-gigabyte or binary files, client-facing documentation and agent prompt templates should reinforce the standard two-stage pattern: guiding agents to inspect EML metadata and entity schemas in-context, and to generate code for out-of-band data retrieval and analysis in local execution environments, rather than attempting to return raw data payloads directly into prompt token context.


## References

- [PASTA+ Core REST API Documentation](https://pastaplus-core.readthedocs.io/en/latest/index.html)
- [PASTA+ Data Package Manager API Documentation](https://pastaplus-core.readthedocs.io/en/latest/doc_tree/pasta_api/data_package_manager_api.html)
- [EDI Data Repository Documentation](https://edirepository.org/)
- [EDI Data Portal](https://portal.edirepository.org/)
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [Anthropic FastMCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- [Ecological Metadata Language (EML) Specification](https://eml.ecoinformatics.org/)
- [FastAPI Framework Documentation](https://fastapi.tiangolo.com/)
- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [HTML Server-Sent Events (SSE) Specification](https://html.spec.whatwg.org/multipage/server-sent-events.html)
- [RFC 7519: JSON Web Token (JWT)](https://datatracker.ietf.org/doc/html/rfc7519)
- [PEP-15: Authentication with API Keys](./pep-15.md)
- [PEP-25: System Activity Logger](./pep-25.md)
- [PEP-27: A "Human" Access Level for use in Authorization](./pep-27.md)


## Rejection

*(To be completed by ETSC if proposal is rejected)*
