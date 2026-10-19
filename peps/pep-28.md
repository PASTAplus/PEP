# PEP-28: Model Context Protocol (MCP) Server for the PASTAPlus REST API

- Author(s): Colin Smith
- Contact: colin.smith@wisc.edu
- Status: Draft
- Type: Application
- Created: 2026-10-13
- Reviewed:
- Final:

## Introduction

The Model Context Protocol (MCP), developed by Anthropic and rapidly adopted across the artificial intelligence ecosystem, is an open standard that allows large language models (LLMs) and autonomous AI agents (such as Claude Desktop, ChatGPT, Cursor, and custom Python-based agentic workflows) to discover and interact securely with external tools, contextual resources, and specialized prompt templates. 

The Environmental Data Initiative (EDI) operates the PASTA+ data repository, providing open, curated scientific and ecological data packages via a comprehensive REST API. While the PASTA+ REST API enables traditional scripted workflows in Python, R, and Java, modern scientific workflows are rapidly shifting toward autonomous agentic synthesis, automated literature-to-data cross-referencing, and intelligent metadata curation.

This proposal describes the design, deployment, governance, and security model for an official **PASTAPlus MCP Server**. By exposing PASTA+ through the MCP standard, EDI will enable AI agents to natively query, inspect, and retrieve scientific data while maintaining strict safeguards against denial-of-service (DoS) attacks, agent runaways, unbuffered memory exhaustion, and unauthorized repository mutations. Furthermore, this document outlines a sustainable, tiered service model that accommodates users across different operational scales—from individual researchers conducting interactive exploratory queries to intensive project workflows running high-frequency automated synthesis—ensuring fair resource allocation while recovering compute and bandwidth costs proportionate to utilization.


## Issue Statement

### The Opportunity: Agentic Data Discovery and Synthesis
Scientific data repositories face an accessibility paradox: while millions of ecological observations, sensor streams, and taxonomy datasets are stored across tens of thousands of data packages in PASTA+, finding, synthesizing, and interpreting these datasets remains labor-intensive. Researchers currently write custom one-off scripts using `requests` or `httr` to query PASTA's Solr index and parse Ecological Metadata Language (EML) XML files. Enabling AI agents to directly interact with PASTA+ via standardized MCP tools and resources will drastically lower the barrier to data discovery, facilitate cross-site ecological synthesis, and assist data authors in validating and curating metadata.

### The Operational Challenge: Bot Traffic, Scrapers, and Repository Strain
EDI has experienced severe operational challenges stemming from aggressive robotic scraping and distributed denial-of-service (DDoS) incidents. These bot incursions have repeatedly degraded repository performance and forced EDI engineering to devote substantial time to refactoring access control systems (e.g., imposing authentication requirements in [PEP-14](./pep-14.md), introducing API keys in [PEP-15](./pep-15.md), and creating intermediate "human" access levels in [PEP-27](./pep-27.md)).

Deploying an AI agent interface without strict architectural boundaries introduces operational risks that EDI's software architecture and systems administration team must mitigate:

1. **Denial-of-Service from Agentic Feedback Loops**: Autonomous agents equipped with tool-calling capabilities often enter iterative reasoning loops. An unconstrained agent can trigger dozens of resource-intensive Solr queries or multi-gigabyte data entity downloads within seconds, overwhelming the PASTA+ server and egress pipelines.
2. **"Agent Breakout" and Repository Mutation Concerns**: Allowing AI agents to invoke mutating operations (such as data package creation/update) creates the risk of unintended repository modifications, corrupted package revisions, or upload queue congestion caused by hallucinated or unvalidated agent actions.
3. **Memory Buffering and Resource Exhaustion**: As highlighted in [PEP-25](./pep-25.md), buffering large XML payloads or data entities entirely in application memory creates high server instability. Any intermediate proxy or server must avoid in-memory buffering when serving multi-megabyte or multi-gigabyte datasets.
4. **Supporting Diverse User Workloads Sustainably**: User workflows interact with the repository at vastly different scales. A casual researcher asking an assistant to locate a few relevant datasets generates minimal load. Conversely, an automated research lab pipeline running cross-site synthesis across hundreds of packages or performing batch metadata congruence evaluations generates intense, sustained traffic. Providing unbounded access to high-throughput agent loops risks degrading service for the broader community, while flat rate-limiting would artificially stifle productive, large-scale scientific applications. A tiered access framework allows users to scale their agentic interactions to match their workflow demands while contributing proportionally to the repository's cloud infrastructure expenses.


## Proposed Solution

To resolve these challenges, we propose a **Hybrid Architecture** for the PASTAPlus MCP Server built with Python and FastAPI, governed by multi-layer bot mitigation and rate-limiting infrastructure on AWS, backed by a phased safety rollout, and monetized through tiered API Gateway usage plans.

```
                           +-------------------------------------------------------+
                           |                     Client Agents                     |
                           |   (Claude Desktop, Cursor, Custom Agent Workflows)    |
                           +---------------------------+---------------------------+
                                                       |
                                      HTTPS (SSE / Streamable HTTP)
                                         Header: X-API-Key: <key>
                                                       v
+---------------------------------------------------------------------------------------------------------+
| AWS Cloud Infrastructure (EDI Managed)                                                                  |
|                                                                                                         |
|   +------------------------------------+          +-------------------------------------------------+   |
|   | AWS WAF                            |          | Amazon API Gateway (HTTP API / REST)             |   |
|   | - IP Reputation & Bot Filtering   |--------->| - API Key Validation (PEP-15 / PEP-27)          |   |
|   | - Rate-based Rule Blocks (DDoS)    |          | - Usage Plans & Token-Bucket Throttling         |   |
|   +------------------------------------+          |   (Community Free, Pro, Enterprise Tiers)       |   |
|                                                   +------------------------+------------------------+   |
|                                                                            |                            |
|                                                                            v                            |
|                                                   +-------------------------------------------------+   |
|                                                   | PASTAPlus MCP Server (FastAPI / FastMCP)        |   |
|                                                   | - Tool Execution & Resource Handling            |   |
|                                                   | - Non-buffering Asynchronous Byte Streaming     |   |
|                                                   +------------+-----------------------+------------+   |
|                                                                |                       |                |
|                                            Read-Through Cache  v                       v REST Proxy     |
|                                                   +----------------+      +-------------------------+   |
|                                                   | ElastiCache    |      | EDI Gatekeeper          |   |
|                                                   | (Redis Cluster)|      | (Reverse Proxy)         |   |
|                                                   +----------------+      +------------+------------+   |
+----------------------------------------------------------------------------------------|----------------+
                                                                                         | Internal Traffic
                                                                                         | (EDI-MCP-Agent/1.0)
                                                                                         v
                                                                            +-------------------------+
                                                                            | PASTA+ Core Services    |
                                                                            | - Data Package Manager  |
                                                                            | - Audit Manager / PEP-25|
                                                                            | - S3 Storage (PEP-11)   |
                                                                            +-------------------------+
```

### 1. Hybrid Architecture: Dual-Distribution Model

The MCP server will be engineered as a single, modular Python codebase using FastAPI and the official Python MCP SDK (`mcp` / `fastmcp`), supporting two deployment modes:

1. **EDI-Hosted Cloud Gateway (AWS Managed)**:
   - Deployed within EDI's AWS environment, exposing an endpoint over Server-Sent Events (SSE) and Streamable HTTP.
   - Fronted by AWS WAF and Amazon API Gateway to provide centralized authentication, usage plan enforcement, bot defense, response caching, and metered billing.
   - Serves web-based AI clients, cloud agents, and commercial consumers without requiring them to install local runtimes.
2. **Open-Source Local Client Package (`edi-pasta-mcp`)**:
   - Distributed via PyPI and GitHub (`pip install edi-pasta-mcp` or `uvx edi-pasta-mcp`).
   - Runs locally on the researcher's machine over standard I/O (`stdio`), integrating seamlessly into desktop applications like Claude Desktop or Cursor.
   - The local client requires the user to input their personal EDI API key (obtained from IAM per [PEP-15](./pep-15.md)), directing requests either to the EDI-hosted gateway or directly to the PASTA+ Gatekeeper with verified human/authenticated credentials.


### 2. Bot Mitigation, Traffic Governance, and DoS Protection

To alleviate the systems administrator's burden and protect the PASTA+ core cluster from automated agent surges, the hosted MCP service employs a multi-tiered defense:

1. **Mandatory EDI API Key Authentication**:
   - In adherence to [PEP-14](./pep-14.md), [PEP-15](./pep-15.md), and [PEP-27](./pep-27.md), anonymous requests are strictly rejected. Every MCP session must present an active EDI API key via the `X-API-Key` header or MCP connection configuration.
   - API keys are verified at the AWS API Gateway layer before any request reaches the MCP application or PASTA+ backend.
2. **Edge Throttling and Token-Bucket Rate Limiting**:
   - AWS API Gateway Usage Plans enforce per-key rate limits (requests per second) and burst quotas. If an agent loops maliciously or enters a runaway execution state, API Gateway immediately sheds the load with HTTP 429 ("Too Many Requests") responses at the AWS edge, ensuring 0% load reaches the PASTA+ backend.
3. **AWS WAF Bot Control & IP Reputation**:
   - AWS WAF sits in front of the API Gateway, applying managed bot control rule sets, blocking known malicious scanning networks, and inspecting request frequency to thwart distributed volumetric scraping attempts.
4. **Agent Traffic Segregation and Header Tagging**:
   - When the MCP server proxies requests to the EDI Gatekeeper, it automatically injects:
     - `User-Agent: EDI-MCP-Agent/1.0 (AWS-Hosted; +https://edirepository.org)`
     - `EDI-Agent-Client-ID: <hashed-user-id>`
   - This header tagging allows the Gatekeeper and Roger Dahl's System Activity Logger ([PEP-25](./pep-25.md)) to immediately distinguish legitimate, authenticated AI agent traffic from illicit scrapers and anonymous web crawlers.
5. **High-Performance Intermediate Caching (Amazon ElastiCache Redis)**:
   - High-frequency, expensive read operations—including Solr searches (`searchDataPackages`), EML metadata documents (`readMetadata`), Dublin Core XML, and evaluation reports—are cached in an in-memory Redis cluster with configurable Time-To-Live (TTL, e.g., 1 hour for searches, 24 hours for immutable package revisions).
   - Identical agent queries are served directly out of Redis in < 2ms without dispatching HTTP requests to Tomcat.


### 3. Phased Capability Scoping & Sandboxing

To address concerns regarding "agent breakouts" and accidental repository corruption, the MCP server will implement a strict phased capability rollout:

| Phase | Operational Scope | Permitted Environments | Guardrails & Safeguards |
| :--- | :--- | :--- | :--- |
| **Phase 1 (Launch)** | **Strictly Read-Only** | Production (`pasta.lternet.edu`) & Staging (`pasta-s.lternet.edu`) | Zero mutation tools exposed. Agents can only search, discover, inspect metadata, read evaluation reports, and download data entities. |
| **Phase 2 (Future)** | **Mutation & Upload** | **Staging Environment Only** (`pasta-s.lternet.edu`) | Data evaluation (`evaluateDataPackage`) and upload (`createDataPackage`, `updateDataPackage`) strictly gated behind Human-in-the-Loop (HITL) approval tokens. Cannot target Production. |

In Phase 1, there is zero risk of an agent altering repository state, overwriting identifiers, or consuming upload processing queues. In Phase 2, agents will be restricted to staging sandboxes and must receive interactive user confirmation before submitting evaluate or upload transactions.


### 4. MCP Primitives: Tools, Resources, and Prompts

The PASTAPlus MCP server will implement the three core primitives of the Model Context Protocol:

#### 4.1 Tools (Actionable Functions)

##### Discovery & Search
- `search_data_packages(query: str, filter_query: Optional[str] = None, start: int = 0, rows: int = 10, sort: Optional[str] = None) -> str`: Executes a Solr search query against PASTA+ metadata (e.g., query by keyword, taxon, geographic bounding box, temporal range, or author). Returns structured JSON search results.
- `list_data_package_scopes() -> List[str]`: Lists all valid scope identifiers (e.g., `edi`, `knb-lter-hfr`, `knb-lter-nin`).
- `list_data_package_identifiers(scope: str) -> List[int]`: Lists all package identifiers within a given scope.
- `list_data_package_revisions(scope: str, identifier: int, filter: Optional[str] = None) -> List[int]`: Lists all available revision numbers for a given package identifier.
- `list_recent_changes(from_date: str) -> str`: Retrieves recent package creations and updates since a specified ISO 8601 timestamp.

##### Metadata & Quality Inspection
- `read_metadata(scope: str, identifier: int, revision: Union[int, str], format: str = "eml") -> str`: Retrieves the package metadata document (supports `eml`, `dublincore`, or formatted summary).
- `read_data_package_report(scope: str, identifier: int, revision: Union[int, str]) -> str`: Retrieves the quality evaluation report generated by PASTA's congruence and quality checker.
- `read_data_entity_names(scope: str, identifier: int, revision: Union[int, str]) -> List[Dict[str, str]]`: Lists all data entities (tables, raster grids, documents) in a package with their entity IDs and filenames.
- `read_data_entity_size(scope: str, identifier: int, revision: Union[int, str], entity_id: str) -> int`: Returns the exact byte size of a data entity before download.
- `read_data_entity_checksum(scope: str, identifier: int, revision: Union[int, str], entity_id: str) -> str`: Returns the SHA-1 checksum for data integrity verification.

##### Data Access (Raw Pass-Through Streaming)
- `read_data_entity(scope: str, identifier: int, revision: Union[int, str], entity_id: str) -> bytes / str`: Streams the complete, raw data entity payload directly through the MCP response.
- `read_data_package_archive(scope: str, identifier: int, revision: Union[int, str]) -> bytes`: Streams the complete zip archive of all data entities and metadata for a package.

##### Citations & Provenance
- `get_citation(scope: str, identifier: int, revision: Union[int, str]) -> str`: Returns the official bibliographic citation string and DOI for a data package.
- `list_data_package_citations(scope: str, identifier: int) -> str`: Lists all known journal publications and external articles citing the dataset.
- `get_provenance_metadata(scope: str, identifier: int, revision: Union[int, str]) -> str`: Retrieves EML provenance metadata tracking upstream and downstream data derivation.

##### Phase 2 Tools (Staging Sandbox Only)
- `evaluate_data_package(eml_xml: str, hitl_token: str) -> str`: Submits an EML document to the PASTA Staging evaluation pipeline (`pasta-s.lternet.edu`) to generate a validation quality report.
- `create_data_package(eml_xml: str, hitl_token: str) -> str`: Submits a new data package to the staging repository after human review.

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


### 5. Data Entity Handling & Full Raw Pass-Through Streaming

In accordance with EDI's operational preferences, the MCP server implements a **Full Raw Pass-Through Streaming** model for data entity retrieval:
- **Zero In-Memory Buffering**: To prevent out-of-memory errors on both the MCP server and PASTA+ nodes (addressing the critical buffering vulnerability highlighted in [PEP-25](./pep-25.md)), the FastAPI MCP server uses asynchronous chunked transfer streaming (`httpx.AsyncClient` streaming piped directly to `StreamingResponse`).
- **Client-Side Context Management**: When an agent invokes `read_data_entity`, the server streams the entire data stream verbatim. The client agent application (Claude Desktop, Cursor, or user Python script) retains full responsibility for managing its LLM context window, buffering limits, or local disk persistence.
- **Header Transparency**: The MCP server forwards `Content-Length`, `Content-Type`, and `Content-Disposition` headers so consuming agents can inspect payload size before committing tokens.


### 6. Sustainable Operational Model & Capacity Tiers

Operating high-throughput AI agent gateways incurs real AWS cloud expenses (compute, NAT gateway bandwidth, S3 data egress, and ElastiCache). To ensure long-term financial sustainability and equitable resource allocation across diverse scientific use cases, the hosted MCP service will offer structured capacity tiers based on workload scale.

#### 6.1 Enforcement Pathways: EDI IAM vs. AWS API Gateway

We identify two architectural pathways for managing service tiers and enforcing usage quotas:

##### Pathway A (Recommended): Native EDI IAM Groups & Decentralized Enforcement
- **Tier Membership in IAM**: Capacity tiers are represented natively as EDI IAM groups (e.g., `mcp-starter`, `mcp-team`, `mcp-power`) assigned to user profiles per [PEP-7](./pep-7.md) and [PEP-9](./pep-9.md).
- **Subscription Lifecycle**: Capacity tiers upgraded via the EDI Data Portal (integrated with a billing processor such as Stripe or institutional recharge) automatically update the user profile's IAM group membership via webhook.
- **Token-Encoded Entitlements**: When the user's API key is validated ([PEP-15](./pep-15.md)), the returned EDI authentication token already carries their group membership. The FastAPI MCP server extracts the tier directly from the token and enforces rate limits and tool access using the local ElastiCache Redis store.
- **Advantages**:
  1. *Single Source of Truth*: Leverages existing EDI user accounts, ORCID logins, and profile management without synchronizing state into external AWS services.
  2. *Domain Granularity*: IAM can automatically assign appropriate tiers to verified research groups, LTER sites, or institutional partners based on profile metadata or grant allocations.
  3. *Vendor Independence*: Avoids cloud lock-in to AWS API Gateway usage plans, ensuring tier management remains portable across hosting environments.

##### Pathway B: AWS API Gateway Usage Plans
- Tiers are configured as native API Gateway Usage Plans with API keys provisioned directly in AWS.
- **Advantages**: Native edge rejection of quota overages.
- **Disadvantages**: Decouples billing/tier status from the EDI user directory, requiring bidirectional synchronization between IAM and AWS API Gateway.

##### Recommended Hybrid Synthesis
We recommend **Pathway A** for authorization and tier policy, paired with **AWS WAF** at the perimeter:
- **EDI IAM** acts as the authoritative registry for user tiers, group memberships, and tool permissions.
- **FastAPI MCP + Redis** enforces per-tier quotas, burst limits, and tool feature flags.
- **AWS WAF / Edge Gateway** provides coarse volumetric bot filtering and DDoS protection, ensuring malicious traffic is dropped before consuming IAM database connections.

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
| **S3 Egress Optimization** | Proxied through PASTA web servlet | Proxied through PASTA web servlet | Direct Presigned S3 URLs ([PEP-11](./pep-11.md)) |
| **SLA & Support** | Best-effort community support | 99.5% uptime SLA, email support | 99.9% uptime SLA, dedicated support channel |
| **Billing Gateway** | N/A | Stripe Billing / EDI Portal recharge | Invoiced / AWS Marketplace SaaS Contract |

#### 6.3 Cost Recovery and Sustainability Impact
By aligning tiers with operational scale rather than institutional identity, EDI ensures that individual researchers and students retain free, frictionless access for interactive exploration. Simultaneously, users operating high-throughput automated synthesis pipelines or institutional batch workflows contribute proportionately to infrastructure costs, turning potential repository strain into a sustainable funding mechanism for EDI's cloud footprint.


### 7. Effects on Related EDI Systems

- **Gatekeeper (Reverse Proxy)**: 
  - Offloaded from handling high-frequency repeated searches due to MCP Redis caching.
  - Receives pre-authenticated requests bearing valid EDI API keys and explicit `EDI-MCP-Agent/1.0` headers, eliminating false-positive bot-blocking triggers.
- **Identity and Access Management (IAM)**:
  - Validates API keys presented by the MCP server per [PEP-15](./pep-15.md) and [PEP-27](./pep-27.md).
  - Acts as the central authority for capacity tier management via IAM groups (`mcp-starter`, `mcp-team`, `mcp-power`). The returned EDI authentication token encodes group memberships, allowing the MCP gateway to enforce tier quotas without additional database queries.
- **Data Package Manager**:
  - Shielded from volumetric query storms by API Gateway rate limiters.
  - Receives well-formed, standard REST calls without needing modification.
- **System Activity Logger ([PEP-25](./pep-25.md)) / Audit Manager**:
  - The distinct `EDI-MCP-Agent/1.0` user-agent and client ID enable clean segregation of AI agent metrics from human portal downloads, preventing statistical skew in dataset usage reports.
- **AWS S3 Storage ([PEP-11](./pep-11.md))**:
  - In Power / High-Volume tiers, large data entity downloads can be served via presigned S3 URLs generated via PEP-11 S3 integration, offloading heavy multi-gigabyte transfers completely from the PASTA application servers.


### 8. Cost & Infrastructure Estimates

| AWS Component | Purpose | Estimated Monthly Cost (USD) |
| :--- | :--- | :--- |
| **Amazon API Gateway (HTTP/REST)** | Request routing, throttling, API key authentication | $30 – $100 (based on 10M–30M calls) |
| **AWS WAF** | Bot Control rules, IP rate limiting, DDoS shielding | $15 – $40 |
| **Amazon ElastiCache (Redis)** | Caching Solr queries, EML XML, and metadata | $40 – $120 (`cache.t4g.small` or `medium`) |
| **Compute (Open Issue below)** | Hosting FastAPI MCP server (ECS Fargate / Lambda / App Runner) | $25 – $80 |
| **AWS Data Egress** | Serving raw data entities to client agents | Variable ($0.09/GB; offset by higher-capacity tiers) |
| **Total Estimated Base Overhead** | | **~$110 – $340 / month** |

*Note: A small number of Team or Power tier subscribers completely covers the monthly infrastructure overhead of the entire MCP service, ensuring that free Starter access remains permanently sustainable for individual researchers.*


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
| **Protocol / Transport** | JSON-RPC 2.0 (stdio / SSE) | Stateless HTTP REST | Python method calls | Stateless HTTP REST |
| **EDI Hosting Footprint** | FastAPI container (or local stdio) | Static file hosting | PyPI package maintenance | FastAPI container |
| **Desktop Client Support** (Claude Desktop, Cursor) | **Native out-of-the-box** | Requires custom bridges | No (requires code sandbox) | Requires custom bridges |
| **Context Token Efficiency** | High (Structured tool schemas) | Low (Monolithic schemas in prompt) | **Very High** (Local in-memory filtering) | High (Compact JSON) |
| **Payload Format Handling** | Handled by server tool wrappers | Raw PASTA+ XML | Parsed to DataFrames/Dicts | Translated to JSON |
| **Support for Resources & Prompts**| **Yes** (Native primitives) | No (Tools only) | No (Tools only) | No (Tools only) |
| **Traffic & Rate Limit Defense** | Enforced at MCP/WAF layer | Gatekeeper / WAF only | Uncontrolled client calls | Enforced at proxy layer |


## Open issue(s)

1. **AWS Compute Runtime Selection**:
   The exact AWS compute runtime for the hosted FastAPI MCP server is left open for evaluation and final selection by the EDI Lead Software Architect and Systems Administrator (Roger Dahl). The primary alternatives under consideration are:
   - **Alternative A: AWS ECS Fargate**: Long-lived Docker containers managed under ECS with an Application Load Balancer. *Pros*: Native, persistent HTTP streaming and Server-Sent Events (SSE) support; zero timeout limits; steady memory footprint; simple VPC integration with existing PASTA services. *Cons*: Persistent baseline hosting cost even during idle periods.
   - **Alternative B: AWS Lambda with Response Streaming**: Serverless Python functions using the AWS Lambda Web Adapter. *Pros*: Scales down to zero cost during idle periods; zero container maintenance. *Cons*: 15-minute execution hard ceiling; potential cold starts; streaming payload size restrictions through API Gateway.
   - **Alternative C: AWS App Runner**: Fully managed container platform connected to Amazon ECR. *Pros*: Automated autoscaling, managed TLS and load balancing with lower operational overhead than ECS. *Cons*: Less flexible VPC peering and custom network configuration.
2. **Human-in-the-Loop (HITL) Token Specification for Phase 2**:
   For Phase 2 evaluation and upload tools, an exact cryptographic token mechanism (e.g., short-lived signed JWTs generated via the EDI Data Portal after human review) must be designed to verify that a human operator explicitly authorized the staging upload.
3. **Billing Integration Platform**:
   Whether commercial subscriptions should be billed via Stripe Billing (integrated into the EDI website/portal) or listed as a SaaS Product on the AWS Marketplace requires evaluation with EDI administrative leadership.
4. **Client-Side LLM Context Windows on Large Datasets**:
   Because full raw pass-through streaming delivers complete datasets directly to the client agent, standard client-side guidelines or helper prompts should be established in documentation to prevent agents from attempting to feed massive binary or multi-gigabyte tabular files directly into token context windows.


## References

- [PEP-0: PEP Purpose and Guidelines](./pep-0.md)
- [PEP-5: Transition from Flask to FastAPI](./pep-5.md)
- [PEP-11: AWS S3 Plan for EDI Data Resources](./pep-11.md)
- [PEP-14: Requiring Authentication for Data Access](./pep-14.md)
- [PEP-15: Authentication with API Keys](./pep-15.md)
- [PEP-25: System Activity Logger](./pep-25.md)
- [PEP-27: A "Human" Access Level for use in Authorization](./pep-27.md)
- [PASTA+ Core REST API Documentation](https://pastaplus-core.readthedocs.io/en/latest/index.html)
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [Anthropic FastMCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)


## Rejection

*(To be completed by ETSC if proposal is rejected)*
