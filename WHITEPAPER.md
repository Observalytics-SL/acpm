# Agent Capability and Profile Model (ACPM): A Vendor-Neutral Standard for Capability, Trust, Cost, and SLA Declaration in Multi-Agent Systems

**Bob van Bussel**
Observalytics SL · bob@observalytics.com
Schema Commons Working Group
June 2026

---

## Abstract

> Orchestrators, registries, and marketplaces operating across multiple AI agents, platforms, tools, and models must make automated dispatch and approval decisions — yet no standard vocabulary exists for the information those decisions require. There is no machine-readable, vendor-neutral way to ask whether a subject supports a given capability at first-class fidelity or only via workaround, what trust evidence backs that claim, what it costs per invocation, or what service-level commitment applies. This paper describes the Agent Capability and Profile Model (ACPM, SC-014), a JSON Schema 2020-12 standard defining a portable **capability profile** document for any AI subject — agent, platform, tool, or model. An ACPM profile declares capability support levels using a four-value enum (native, emulated, partial, none) against a dot-namespaced vocabulary aligned with SC-006 AAIF's `required_capabilities[]` field, enabling direct requirement-to-offering matching without translation. The profile further declares a five-tier trust and attestation posture, a cost block covering pricing model and unit cost, service-level commitments (uptime, latency percentiles, support tier), structured delegation rules reusing AAIF's Condition type, and a compliance and provenance block. ACPM defines five cumulative conformance levels (Basic through Enterprise) and is implemented as a strict JSON Schema with three validated examples spanning agent, platform, and model subjects. ACPM is part of Schema Commons (CC BY 4.0), positioned as the profile/comparison layer complementing SC-006 AAIF (agent definition portability) and SC-013 ARP (agent discovery). No existing effort — including A2A Agent Cards, MCP Server Cards, HACP, or OWASP AgBOM — combines capability fidelity, trust attestation, cost, SLA, and delegation across all four subject types in a single, vendor-neutral document.

---

**Keywords:** AI agents, capability profile, trust and attestation, capability negotiation, cost comparison, service level agreement, multi-agent systems, JSON Schema, Schema Commons, interoperability, agent orchestration, vendor-neutral standard

---

## 1. Introduction

Multi-agent AI systems increasingly depend on runtime decisions that require structured, comparable information about the subjects being composed. When an orchestrator must route a task to one of several available agents, when a marketplace user compares competing platforms, or when a security reviewer approves a new tool for enterprise deployment, the same questions recur: what capabilities does this subject actually support, and at what fidelity? What trust evidence backs the published claims? What does it cost per invocation, and what uptime and latency commitment applies? Under what conditions may it delegate work further, and to whom?

Today, none of these questions has a standard answer. Capability claims live in marketing pages and free-text documentation. Trust is expressed, if at all, through informal badges or proprietary audit certificates that no common schema can read. Cost information requires calling each vendor's API or reading bespoke pricing pages. SLA commitments are buried in service agreements rather than accessible to the orchestration layer. Delegation permissions are hardcoded into orchestrator logic rather than declared by the subject itself. The result is that every registry, marketplace, and orchestration platform writes its own per-vendor integration code, and the capability matching, trust gating, and cost-optimised routing that automated multi-agent systems require either happens ad hoc or does not happen at all.

This fragmentation is not unique to AI agents. Cloud infrastructure faced an analogous problem before provider-neutral service catalog schemas emerged: a resource's capabilities, pricing, and SLA were readable only through the provider's own management console. Hardware platforms faced a comparable problem before standardised capability registers allowed firmware to query what a CPU or peripheral actually supports at runtime. OpenAPI server metadata addressed a narrower version of the same problem for HTTP services, enabling tooling to reason about endpoints without reading human documentation. In each case, a lightweight, vendor-neutral declaration format — not a protocol, not an enforcement mechanism — was sufficient to unlock ecosystem-level tooling.

ACPM occupies this position for the AI agent layer. It is a declaration format, not an execution protocol and not an enforcement mechanism. A capability profile is a claim a subject makes about itself; whether that claim is verified is the consuming platform's responsibility. The standard specifies the shape of the claim precisely enough that two profiles from different vendors can be compared mechanically, that an orchestrator can compute the set of capabilities a candidate subject is missing relative to an agent's declared requirements, and that a security policy can gate on trust level without reading vendor-specific documentation.

The broader context matters here. The companion standard SC-006 AAIF [5] addresses the *definition portability* problem: how to describe an agent once and run it on any platform. SC-013 AREG [12] addresses the *discovery* problem: how to list and query agents in a registry. ACPM addresses a third, orthogonal layer: the *capability/trust/cost comparability* problem. These three standards compose without coupling. An AAIF agent declares `required_capabilities[]` — the dot-namespaced capabilities it needs. An ACPM profile declares `capabilities[]` — the dot-namespaced capabilities a subject offers, each with a support level. Both arrays share the same vocabulary by design, so a capability matching operation is a set subtraction, not a translation problem. An AREG registry entry may reference an ACPM `profile_id` so that a registry query can return structured capability and cost data without the registry itself carrying that data.

The contribution of ACPM is the combination: a single document format, applicable uniformly across agents, platforms, tools, and models, that carries capability fidelity, trust attestation, cost, SLA, delegation rules, and compliance posture in a strict, extensible JSON Schema. The remainder of this paper describes the motivation in detail (Section 2), the design principles that shaped the standard (Section 3), the full data model (Section 4), the conformance levels (Section 5), the relationship to related work (Section 6), the validation approach (Section 7), a discussion of adoption path and known limitations (Section 8), and a conclusion (Section 9).

---

## 2. Background and Problem Statement

### 2.1 What "Capability" Means Today

The term "capability" is used loosely across the current generation of AI agent standards and platforms. The Agent2Agent (A2A) protocol [6] includes an Agent Card with a minimal set of capability tags, but these are free-form strings with no standardised support level — an agent either lists a capability or does not, with no distinction between first-class support and a workaround implementation. The Model Context Protocol [7] defines a capability negotiation handshake in its `initialize` message, but this is a runtime protocol exchange for a live MCP connection, not a static, publishable description; it is also scoped to MCP servers and does not generalise to agents, platforms, or underlying models. AAIF's `required_capabilities[]` field [5] establishes a dot-namespaced vocabulary (`tool.mcp`, `memory.vector`, `orchestration.parallel`, and so on) for declaring what an agent *needs*, but no standard exists for declaring what a subject *offers* at a declared fidelity level.

The practical consequence is that when an orchestrator needs to decide whether a candidate platform can host an agent that requires `memory.vector` and `orchestration.parallel`, the orchestrator must either call the platform's proprietary API, read its documentation, or simply try and observe failure. There is no standard field to read, no vocabulary to match against, and no fidelity qualifier to distinguish "we have a vector memory backend" from "we route vector queries through an in-memory approximation."

### 2.2 What Is Missing

Four specific gaps are architecturally significant for automated multi-agent systems:

**Capability support level.** Binary presence/absence is insufficient. An agent that requires `tool.openapi` may fail silently on a platform that supports OpenAPI spec parsing but not callback-based operations — a `partial` support declaration in ACPM terms. An orchestrator routing on binary capability tags cannot detect this mismatch. The four support levels in ACPM (`native`, `emulated`, `partial`, `none`) are the minimum vocabulary needed to express whether a capability is first-class, approximated, limited, or absent — distinctions that matter for routing decisions and for correctly setting user expectations.

**Trust attestation.** The current practice is for orchestrators to hardcode a trusted list of platforms or to rely on API key possession as a proxy for trust. Neither approach scales to marketplace or multi-tenancy scenarios. There is no standard way for a subject to declare who audited it, by what method, with what evidence, and with what expiry — and no standard vocabulary for a consumer to gate on that declaration. ACPM's five-tier trust model and attestation block provide this vocabulary without prescribing a verification protocol.

**Cost comparability.** Cost information for AI platforms and models is currently available only through vendor-specific pricing pages or proprietary cost APIs. An orchestrator that needs to select the lowest-cost capable agent among several candidates must integrate with each vendor's billing API separately. A standard `cost_profile` block with a closed `pricing_model` enum and a normalized `unit_cost` makes cross-vendor cost comparison a data operation rather than an integration project.

**SLA-based routing.** Service-level data — uptime percentages, response latency percentiles, support tier — is similarly siloed. An orchestrator enforcing a latency SLA across a multi-agent pipeline currently has no standard source for the per-component latency data it needs. ACPM's `sla` block makes that data readable from a profile rather than from a vendor agreement.

### 2.3 Failure Scenarios

Three specific failure modes in current practice motivate the standard directly:

First, **silent capability mismatch**: an agent requires `tool.openapi`, a platform declares it in free-text documentation, the agent is deployed, and callback-based operations fail at runtime. An ACPM `partial` declaration with an explanatory `notes` field would have made this mismatch detectable at dispatch time.

Second, **opaque cost**: two candidate platforms both support the required capabilities, but selecting between them on cost requires separate API calls to each vendor's billing endpoint, with different authentication, different response schemas, and different billing granularities. An ACPM `cost_profile` makes this comparison a local data operation on two JSON documents.

Third, **ad hoc trust gating**: the orchestrator enforces a policy that only `verified` or higher platforms may be dispatched high-sensitivity tasks. This policy today lives as hardcoded logic in the orchestrator, checked against a manually maintained list of approved platforms. An ACPM `trust.level` field makes the policy checkable against a standard field on any profile, and an `attestation.expires_at` check prevents reliance on stale attestations.

---

## 3. Design Principles

Six design principles shaped the standard and are worth making explicit, as they explain several non-obvious choices in the data model.

**Claims, not contracts.** An ACPM profile is a self-description, not a cryptographically enforced contract. ACPM specifies the *shape* of trust and cost claims without prescribing verification protocols. This is a deliberate scope boundary: building a PKI-style verification system into the standard would make it far harder to adopt, while the declaration layer is independently useful — a verified claim from a reputable issuer is worth more than an unverified one, and a consuming platform can choose the level of independent verification it requires. The standard is honest about this: it states explicitly that `trust.level` is claimable without external verification, and that `provenance.signature` is advisory unless a consumer independently checks it.

**One schema for four subject types.** Agents, platforms, tools, and models share the same schema rather than having separate schemas for each type. The practical motivation is that a consumer comparing a tool's ACPM profile to a platform's ACPM profile should not need separate parsing logic for each. A platform that runs agents has a `subject_type: "platform"` profile; the agent running on it has a `subject_type: "agent"` profile; the tool the agent calls has a `subject_type: "tool"` profile; the model the agent routes to has a `subject_type: "model"` profile. All four are readable with the same code and comparable against each other on any shared dimension — capability, trust, cost, SLA.

**Vocabulary-compatible with AAIF.** The `capabilities[].id` dot-namespace is the same vocabulary as AAIF's `required_capabilities[]`. This is not a schema coupling — the two standards reference each other only through `sc_refs[]` cross-references and documentation, not through `$ref` imports. But by convention, an AAIF agent's requirements and an ACPM profile's offerings share the same string vocabulary, so the matching operation `needed_capabilities - offered_capabilities` requires no translation layer.

**Comparable by construction.** Every offer-side concept uses a closed vocabulary rather than free text, specifically so that two profiles can be compared programmatically. The four support levels, five trust tiers, five pricing models, and four billing units are enums, not strings. A consuming platform can rank or filter profiles on any of these dimensions without natural-language parsing.

**Strict by default, extensible at the edges.** `additionalProperties: false` is set throughout the schema. This prevents silently ignored fields that could create false impressions of profile completeness. The `x-` prefix and `_comment` root key provide an escape hatch for vendor-specific metadata and human annotations, respectively, without relaxing validation on the normative fields.

**Composable, not coupled.** ACPM cross-references SC-006 and SC-013 through `sc_refs[]` entries in the subject block. These are data references, not schema dependencies. ACPM can be used without any other Schema Commons standard, and the other standards do not require ACPM.

---

## 4. The ACPM Data Model

An ACPM document — a **capability profile** — is a JSON object conforming to `profile.schema.json` (JSON Schema 2020-12). Two fields are required: `sc_standard` (the literal `"SC-014"`) and `subject` (with `subject_type` and `name`). All other blocks are optional, enabling the conformance level progression described in Section 5. The object model is:

```
CapabilityProfile (root)
├── sc_standard        "SC-014"               [required]
├── sc_version         semver string
├── profile_id         UUID
├── subject                                   [required]
│   ├── subject_type   agent|platform|tool|model
│   ├── name / vendor / version / homepage
│   └── sc_refs[]      [{standard, id}]
├── capabilities[]     [{id, support, version_constraint, notes}]
├── tools[]            [{protocol, max_concurrent_calls, auth_methods}]
├── models[]           [{provider, model_id, context_window, modalities}]
├── memory[]           [{backend, scopes}]
├── trust
│   ├── level          untrusted|sandboxed|verified|attested|enterprise
│   ├── attestation    {issuer, method, evidence_url, issued_at, expires_at}
│   └── sandbox        {isolation, network_egress}
├── cost_profile
│   ├── pricing_model  free|usage_based|subscription|tiered|custom
│   ├── currency / unit_cost / free_tier
├── sla
│   ├── uptime_pct / p50_latency_ms / p99_latency_ms
│   └── support_tier / incident_response
├── delegation_rules[] [{direction, target, condition, action}]
├── compliance         {data_residency[], certifications[], pii_handling}
└── provenance         {authored_by, created_at, updated_at, license, signature}
```

### 4.1 Subject Identity

The `subject` block identifies what the profile describes. `subject_type` is a closed enum: `agent`, `platform`, `tool`, or `model`. `name` is the human-readable label. `vendor`, `version`, and `homepage` are optional metadata. The most structurally significant optional field is `sc_refs[]`, an array of `{standard, id}` pairs that link the profile into the rest of the Schema Commons ecosystem. A profile describing an agent typically carries `{"standard": "SC-006", "id": "<agent_id>"}` pointing to the corresponding AAIF agent definition; a profile describing a platform listed in an AREG registry carries `{"standard": "SC-013", "id": "<registry_entry_id>"}`. These cross-references allow a consumer that begins at an AREG registry entry to follow a reference to the ACPM profile and obtain capability/trust/cost data without the registry itself having to carry it, and allow an orchestrator evaluating an AAIF agent to follow a reference to the platform's ACPM profile to verify that the platform supports everything the agent needs.

### 4.2 Capability Declarations

The `capabilities[]` array is the semantic core of the standard. Each entry carries:

- `id` — a dot-namespaced capability string matching the pattern `^[a-z][a-z0-9_]*(\.[a-z][a-z0-9_]*)*$`. ACPM does not enumerate a closed set of capability identifiers; instead, the vocabulary is community-maintained and intentionally aligned with AAIF's `required_capabilities[]` namespace. Current examples include `tool.mcp`, `tool.http`, `tool.function`, `tool.openapi`, `memory.vector`, `memory.redis`, `memory.postgres`, `orchestration.parallel`, `orchestration.pipeline`, `orchestration.handoff`, `orchestration.consensus`, `vision.image_understanding`, `audio.speech_to_text`, `reasoning.extended_thinking`, `auth.vault`, `state.checkpoint`, `state.restore`, `telemetry.otel`, `compliance.hitl`, `compliance.audit_log`, `io.streaming`, and `io.async`.

- `support` — one of `native`, `emulated`, `partial`, or `none`. **Native** means first-class, fully integrated support. **Emulated** means the capability is achieved through composition or workaround — it works but may carry performance, fidelity, or maintenance caveats. **Partial** means the capability is supported with material limitations that a consumer should read in `notes` before relying on it. **None** is an explicit declaration of absence, useful for comparison (a consumer can distinguish "not declared" from "declared absent"). The four-value enum is the minimum needed to express the distinctions that matter for automated routing; a binary present/absent flag conflates cases that orchestrators need to distinguish.

- `version_constraint` — an optional semver range over the subject's own version for which the claim holds, enabling profiles to express "this capability is available from version 4.0.0 onwards."

- `notes` — free-form text for caveats, links to documentation, or clarification of a `partial` or `emulated` declaration.

### 4.3 Inventory

The `tools[]`, `models[]`, and `memory[]` arrays describe what the subject works *with*, complementing the capability declarations.

`tools[]` lists the tool-invocation protocols the subject supports (`function`, `mcp`, `http`, `openapi` — aligned with AAIF's `tools[].protocol`), with optional `max_concurrent_calls` and `auth_methods` per protocol. This makes the operational limits of a subject's tool interface readable from the profile rather than requiring a runtime negotiation to discover them.

`models[]` lists the LLMs the subject exposes, routes to, or (for `subject_type: "model"`) is itself. Each entry carries `provider`, `model_id`, `context_window`, `max_output_tokens`, and `modalities`. This is complementary to but distinct from OWASP AgBOM [10] supply-chain provenance: AgBOM inventories what an agent is *built from* for vulnerability and license tracking; ACPM's `models[]` inventories what LLMs a subject *currently exposes* for capability-matching and routing purposes.

`memory[]` lists supported storage backends (`in_memory`, `redis`, `postgres`, `sqlite`, `pinecone`, `weaviate`, `qdrant`, `chroma`, `custom`) and the scopes available on each (`user`, `session`, `task`, `long_term`), using the same vocabulary as AAIF's `memory[]` block.

### 4.4 Trust and Attestation

The `trust` block carries three sub-components.

`trust.level` is a five-tier ordered enum representing increasing assurance:

| Level | Meaning |
|-------|---------|
| `untrusted` | No claims verified by any party. |
| `sandboxed` | Subject executes under an isolation boundary (see `trust.sandbox`), but capability and cost claims are unverified. |
| `verified` | Claims checked — by the publisher or informally — against observed behaviour. |
| `attested` | A named issuer has produced a dated, linkable attestation using a declared method. |
| `enterprise` | Attested, plus organisational accountability typically evidenced by `sla.support_tier` and `compliance` being populated. |

`trust.attestation` carries the evidence backing the claimed level: `issuer` (the attestation authority), `method` (`self`, `third_party`, or `formal_audit`), `evidence_url` (a link to the supporting document), and `issued_at`/`expires_at` timestamps. A consumer gating on `attested` or `enterprise` trust SHOULD verify that `expires_at`, if present, has not passed.

`trust.sandbox` describes the isolation boundary the subject runs within: `isolation` (`none`, `process`, `container`, `vm`) and `network_egress` (`none`, `allowlist`, `full`). This is the subject's self-declared execution environment, not the isolation a remote caller receives — transport-level security is the consuming platform's responsibility.

The trust model is explicitly self-declared. ACPM places no technical barrier against an inflated trust claim. The standard's position is that specifying the *shape* of a trust claim is independently useful — it makes trust information readable and comparable — even in the absence of a verification protocol. A consuming platform that needs higher assurance SHOULD independently verify `attestation.evidence_url` and, if present, `provenance.signature`. The `provenance.signature` field is a base64-encoded signature over the canonical JSON of the profile document; ACPM does not mandate a signature scheme or key-distribution mechanism, consistent with the standard's scope boundary of declaring rather than enforcing.

### 4.5 Cost Profile

The `cost_profile` block enables cross-vendor cost comparison as a local data operation. `pricing_model` is a closed enum (`free`, `usage_based`, `subscription`, `tiered`, `custom`). `unit_cost.per` normalises the billing dimension to one of `request`, `token_1k`, `minute`, or `run`, and `unit_cost.amount` states the price in `currency` (ISO 4217). `free_tier` optionally declares a number of included units per period before billing applies.

The motivation for cost as a first-class block is direct: an orchestrator that needs to select the lowest-cost capable agent among several candidates can read `unit_cost` from each profile and compare locally, without calling each vendor's billing API. A FinOps review of an agent fleet can compare `cost_profile` blocks across all registered subjects without requiring per-vendor integrations. The schema is intentionally narrow — one billing dimension, no tiered pricing table — because simplicity enables comparison. Multi-dimensional pricing should be published as multiple profiles per tier, with full detail at `subject.homepage`.

### 4.6 Service Level Agreements

The `sla` block carries `uptime_pct`, `p50_latency_ms`, `p99_latency_ms`, `support_tier`, and `incident_response.severity1_minutes`. These fields may represent either contractual commitments or observed historical figures; the standard does not currently distinguish the two (a known gap, discussed in Section 8).

SLA data in a standard profile enables SLA-based routing at the orchestration layer. An orchestrator enforcing an end-to-end latency budget can read `p99_latency_ms` from each candidate subject's profile and select the combination that satisfies the budget. A platform selection policy that requires `support_tier: "enterprise"` for production deployments can check that field without reading each vendor's support agreement. Neither operation requires a live API call to the vendor.

### 4.7 Delegation Rules

The `delegation_rules[]` array governs whether the subject may hand work to (`outbound`) or accept work from (`inbound`) another subject. Each rule is a `{direction, target, condition, action}` object.

`target` optionally scopes the rule to a specific counterparty by `profile_id` (UUID), to a specific capability by `capability_id`, or to a minimum counterparty trust level by `trust_level_min`. `action` is `allow`, `deny`, or `require_approval`.

The `condition` field reuses the **Condition** type defined in SC-006 AAIF (SPECIFICATION §V), copied verbatim into ACPM's schema `$defs`. The structured form — `{"lang": "cel"|"jsonlogic"|"jsonpath"|"jmespath"|"always"|"never", "expr": ...}` — is deterministic and portable: every conforming consumer evaluates it identically. The string form is a natural-language hint only and MUST NOT be relied on for portable gating. When a consumer does not support the declared `lang`, it MUST treat the rule as unsatisfiable-unknown and apply a conservative default (deny for outbound, reject for inbound). `require_approval` MUST be honoured; treating it as `allow` is a conformance violation.

The motivation for structured delegation rules in the profile — rather than in orchestrator logic — is auditability. When delegation permissions are declared in the profile, a security review of a multi-agent system can read the delegation graph from the profiles rather than reverse-engineering it from orchestrator code. The use of the same Condition type as AAIF means that the expression languages available for routing conditions in AAIF are the same ones available for delegation gating in ACPM, reducing the cognitive and tooling surface for implementers familiar with both standards.

### 4.8 Compliance

The `compliance` block carries `data_residency[]` (regions where the subject can guarantee data is stored/processed: `EU`, `US`, `UK`, `APAC`, `custom`), `certifications[]` (free-form strings such as `"SOC2"`, `"ISO27001"`, `"HIPAA"`), and `pii_handling` (`none`, `redact`, `encrypt`, `forbidden`).

The relationship to AAIF's compliance block is a useful distinction: AAIF's `compliance` declares the *environment* an agent requires — the data residency and PII handling that a platform must provide for the agent to be deployed there. ACPM's `compliance` declares the subject's *published posture* — the data residency it can guarantee and the PII handling it performs. These are offer-side and requirement-side declarations of the same concepts, and they are designed to be matched against each other: a platform profile with `data_residency: ["EU"]` satisfies an AAIF agent that requires `data_residency: "EU"`.

### 4.9 Illustrative Example

The following excerpt from the Invoice Chaser agent profile (`examples/invoice-chaser-profile.json`) illustrates a complete `agent`-type profile at the Priced conformance level:

```json
{
  "sc_standard": "SC-014",
  "sc_version": "1.0.0",
  "profile_id": "b2c3d4e5-0001-4000-8000-profile000001",
  "subject": {
    "subject_type": "agent",
    "name": "Invoice Chaser",
    "vendor": "Rivera Consulting",
    "version": "1.2.0",
    "sc_refs": [
      { "standard": "SC-006", "id": "a1b2c3d4-0001-4000-8000-invoice000001" }
    ]
  },
  "capabilities": [
    { "id": "tool.mcp",  "support": "native" },
    { "id": "tool.http", "support": "native" },
    { "id": "memory.postgres", "support": "native" },
    { "id": "memory.vector",   "support": "none" },
    { "id": "orchestration.parallel", "support": "none" },
    { "id": "auth.vault", "support": "emulated",
      "notes": "Resolves secrets via env vars today; vault_ref planned." }
  ],
  "trust": {
    "level": "verified",
    "attestation": { "issuer": "Rivera Consulting", "method": "self",
      "issued_at": "2026-06-10T09:00:00Z" },
    "sandbox": { "isolation": "container", "network_egress": "allowlist" }
  },
  "cost_profile": {
    "pricing_model": "usage_based", "currency": "USD",
    "unit_cost": { "per": "run", "amount": 0.04 },
    "free_tier": { "included_units": 100, "period": "month" }
  },
  "delegation_rules": [
    { "direction": "inbound",
      "target": { "capability_id": "orchestration.handoff",
                  "trust_level_min": "verified" },
      "condition": { "lang": "always" }, "action": "allow" },
    { "direction": "outbound",
      "target": { "trust_level_min": "verified" },
      "action": "require_approval" }
  ]
}
```

This profile is immediately machine-readable by an orchestrator: it knows the agent does not support parallel orchestration (`"none"`) and uses only emulated vault authentication, that it runs in a container with allowlist egress, that inbound delegation from any verified-or-higher subject is unconditionally allowed, and that outbound delegation requires approval. An orchestrator can extract and act on all of these facts without calling any vendor API.

---

## 5. Conformance Model

ACPM defines five cumulative conformance levels. Each level is a predicate over the profile document; a profile satisfying level N satisfies all predicates of levels 1 through N-1 as well.

| Level | Requirements |
|-------|-------------|
| **Basic** | Validates against `profile.schema.json`; `subject` present with `subject_type` and `name`. |
| **Tooled** | Basic + `capabilities[]` is non-empty. |
| **Trusted** | Tooled + `trust.level` is one of `verified`, `attested`, or `enterprise`. |
| **Priced** | Trusted + `cost_profile` is present. |
| **Enterprise** | Priced + `sla` present + `compliance` present + `provenance.signature` present. |

The progression logic reflects real deployment requirements. A **Basic** profile is immediately useful for discovery — it tells a registry what the subject is and who publishes it. A **Tooled** profile enables automated capability matching. A **Trusted** profile is the minimum for security-gated orchestration. A **Priced** profile enables cost-optimised routing. An **Enterprise** profile provides everything a regulated deployment requires for due diligence: verified trust, cost transparency, SLA commitments, compliance posture, and a tamper-evident signature over the profile document.

Conformance is **self-certified**. There is no central authority that issues conformance certificates. This is the same model used by AAIF [5], by HTML5's "valid HTML" framing, and by most successful open standards at the declaration layer. The practical rationale is that gatekeeping conformance centralises a bottleneck that would prevent adoption; the value of the standard comes from broad adoption of a common schema, not from certification. The self-certification tooling — a machine-readable conformance report format analogous to AAIF's `conformance-report.schema.json` — is specified in prose in CONFORMANCE.md but not yet shipped as a schema artefact for ACPM v1.0. This is a planned gap, not a design omission, and is flagged as a priority item before a stable release.

---

## 6. Related Work

### 6.1 A2A Agent Cards

The Agent2Agent (A2A) protocol [6] defines an Agent Card as a lightweight, standardised metadata document that an agent publishes to describe its identity, capabilities, and supported interaction modes. ACPM and A2A Agent Cards share the goal of machine-readable agent discovery, but differ substantially in scope. An Agent Card carries a minimal set of free-form capability tags and identifies the agent's API endpoint; it does not define support levels for capabilities, does not carry trust attestation or a sandbox declaration, does not carry a pricing model or unit cost, and does not carry SLA commitments or delegation rules. ACPM extends the Agent Card concept across all four subject types and adds the structured capability fidelity, trust, cost, and SLA dimensions that automated orchestration decisions require. An ACPM `agent`-type profile should be designed to be compatible with or exportable to an A2A Agent Card for the fields they share in common.

### 6.2 MCP Server Capability Negotiation and Server Cards

The Model Context Protocol [7] defines a capability negotiation handshake in its `initialize` request/response: a client declares its capabilities and a server responds with its own, enabling the session to operate at the intersection. This is a runtime protocol exchange for a live MCP connection, not a static publishable document. The MCP development roadmap [8], dated 2026-03-05, describes a planned **MCP Server Cards** effort — "a standard for exposing structured server metadata via a `.well-known` URL, so browsers, crawlers, and registries can discover a server's capabilities without connecting to it." This is the closest in-progress effort to ACPM's `tool`-type profile use case.

The relationship is one of general case and specific instance rather than competition. MCP Server Cards are scoped to MCP servers; ACPM covers agents, platforms, tools, and models across all protocols. ACPM's `tool`-type profile adds trust attestation, cost, SLA, and delegation dimensions that a server card is not designed to express. An ACPM `tool`-type profile describing an MCP server should be designed so that its capability- and protocol-relevant fields are exportable to, or trivially mappable onto, a future Server Card, avoiding redundant dual-maintenance. Because the MCP Server Card effort was still forming at the time of this writing, a compatibility mapping is necessarily provisional and should be re-evaluated before ACPM finalises any concrete interoperability claim.

### 6.3 HACP

Draft-sunyi-hacp-protocol-00 [9] ("HACP: A Capability-Contract Protocol for AI Agents and Edge Hardware") is an IETF individual submission that defines a JSON-RPC 2.0 protocol for LLM agents controlling physical edge hardware — GPIO, I2C, UART, and sensor interfaces — through a security-checked daemon. Each tool registered in HACP carries a `risk_level` from 0 to 3, and a session is bounded by a configured maximum risk level: a session "MUST NOT execute a tool whose risk_level exceeds the session's configured maximum."

HACP is cited here as a genuine, narrow precedent for capability-contract thinking. The underlying idea — declare what a subject can do, and at what risk/trust tier, before relying on it — is conceptually identical to ACPM's `capabilities[]` + `trust.level` combination. However, HACP is scoped exclusively to hardware control, uses a static per-session risk-level cap rather than ACPM's richer per-capability support levels, and carries no cost, SLA, or delegation model. It is a complementary, domain-specific precursor rather than a general-purpose alternative.

### 6.4 OWASP AgBOM

The OWASP Agent Observability Standard project [10] proposes an Agent Bill of Materials (AgBOM) concept, inventorying what an agent or AI system is built from — models, tools, dependencies, configuration — using established software supply-chain formats (CycloneDX, SPDX, SWID). AgBOM shares surface-area overlap with ACPM's `tools[]` and `models[]` inventory blocks.

The distinction is in centre of gravity. AgBOM's purpose is supply-chain provenance: what is *inside* this system, for vulnerability tracking, license compliance, and dependency auditing. ACPM's purpose is runtime offering: what this system can *do*, and under what trust and cost terms it can be relied on. These are complementary rather than competing. A comprehensive description of a production AI system might include both an ACPM profile (capability/trust/cost declaration) and an AgBOM document (supply-chain inventory); ACPM's `subject.sc_refs[]` is designed to carry a cross-reference to such a companion document.

### 6.5 SC-006 AAIF and SC-013 AREG

Within Schema Commons, ACPM occupies the profile/comparison layer between AAIF's definition portability layer [5] and AREG's discovery layer [12]. AAIF defines how an agent is *run* — goals, model routing, orchestration topology, tools, memory, guardrails, runtime policy. AREG defines how an agent is *discovered* in a registry — listing, querying, and filtering registered subjects. ACPM defines what a subject *offers* — capability fidelity, trust, cost, SLA, delegation.

The vocabulary alignment between AAIF and ACPM is intentional and non-trivial: AAIF's `required_capabilities[]` and ACPM's `capabilities[].id` use the same dot-namespaced strings. This means that the operation "does this platform support everything this agent needs?" is computable as a set subtraction on two lists of dot-namespaced strings, without any translation layer. The delegation `condition` type in ACPM is identical to AAIF's Condition type (SPECIFICATION §V), copied verbatim, so the same expression languages and evaluation semantics apply across both standards.

### 6.6 NIST AI Risk Management Framework

The NIST AI Risk Management Framework [11] addresses AI trustworthiness at the organisational governance and process level: risk identification, measurement, management, and governance across the AI lifecycle. ACPM addresses trustworthiness at the per-subject declaration level: a specific agent, platform, tool, or model declaring its trust tier, attestation method, and sandbox configuration. The two are complementary at different granularities. An organisation implementing the NIST AI RMF may find ACPM profiles useful as a data source for per-subject risk assessment and inventory, while ACPM's trust model does not substitute for the organisational governance structures the NIST framework describes.

### 6.7 Open Cybersecurity Schema Framework

The Open Cybersecurity Schema Framework (OCSF) [13] established the pattern that ACPM generalises: a single, vendor-neutral schema that disparate vendors use to describe the same class of data (security events, in OCSF's case) in a comparable, normalised form. ACPM applies the same pattern to AI agent capabilities: rather than each platform describing its capabilities in a proprietary format, all subjects use the same ACPM schema, making capabilities comparable across vendors by construction. The OCSF analogy — "ACPM is to AI agent capabilities what OCSF is to security events" — captures the design intent precisely.

---

## 7. Validation and Implementation

The normative schema artefact is `schema/profile.schema.json`, implemented as JSON Schema draft 2020-12. The schema root sets `"additionalProperties": false` and `"required": ["sc_standard", "subject"]`. The `condition` type is defined in `$defs.condition` as a `oneOf` between a bare string (natural-language hint) and a structured object with a closed `lang` enum plus an `allOf` constraint requiring `expr` for non-constant languages. Pattern constraints enforce UUID format on `profile_id` and `target.profile_id`, semver format on `sc_version` and `subject.version`, and dot-namespace format on `capabilities[].id`.

Three example profiles are included and all validate cleanly against the schema using the Schema Commons reference validator (`tools/validate.py`):

- `examples/invoice-chaser-profile.json` — `subject_type: "agent"`, trust level `verified`, usage-based per-run pricing, EU data residency. Demonstrates the `sc_refs[]` cross-reference to an AAIF agent definition and the explicit `"none"` declarations for capabilities the agent does not support.

- `examples/langgraph-platform-profile.json` — `subject_type: "platform"`, trust level `attested` (third-party, with evidence URL and expiry), tiered pricing per minute, SOC2/ISO27001 certifications, a CEL-language delegation condition gating outbound `tool.http` calls on `network_egress != 'full'`. Demonstrates the full breadth of capability declarations (20 entries spanning tool, memory, orchestration, telemetry, compliance, auth, state, io, and reasoning namespaces) and the `provenance.signature` field required for Enterprise conformance.

- `examples/gpt4o-model-profile.json` — `subject_type: "model"`, trust level `enterprise`, usage-based per-thousand-token pricing, multimodal capability declarations. Demonstrates a model-type profile with no `sc_refs[]` (the model is not described by an AAIF agent definition) and explicit `"none"` for capabilities such as `audio.speech_to_text` and `reasoning.extended_thinking`.

Together, three of the four subject types (agent, platform, model) are covered by the included examples; a `tool`-type profile is not yet included but is structurally identical to the agent-type profile with `subject_type: "tool"`.

What schema validation proves is structural correctness: a profile that validates contains the required fields, uses only values from the closed enums, and satisfies the pattern constraints. What it does not prove is claim accuracy — a validating profile with `trust.level: "enterprise"` and fabricated `attestation.evidence_url` is structurally correct. This is the intended scope boundary. Self-certified conformance declarations and a planned conformance report schema (not yet shipped for ACPM v1.0) address the declaration layer; claim accuracy is the consuming platform's responsibility, as described in Section 4.4.

---

## 8. Discussion

### 8.1 Adoption Path

ACPM is most valuable when the consumers of profiles — registries, orchestrators, marketplaces, security review workflows — adopt it as a standard input. A registry implementing SC-013 AREG [12] that stores ACPM `profile_id` references in its listing entries gains the ability to return structured capability and cost data in listing queries without embedding that data in the registry schema itself. An orchestrator that implements AAIF capability matching can extend the same matching logic to platform selection, gating dispatch on both capability set coverage and `trust.level` minimum. A security review workflow that checks `trust.attestation.expires_at` and `compliance.data_residency` against a deployment policy becomes stateless — it reads the profile rather than querying multiple vendor endpoints.

None of this ecosystem exists today. ACPM v1.0.0 is a **Proposed** standard with no implementations and no adopters. It is a pre-condition for automated multi-agent marketplace behaviour, not a description of current practice. The value of publishing the standard now is to establish the vocabulary and schema before each platform invents its own, making future convergence cheaper. Early adopters who publish ACPM profiles for their platforms or agents are invited to register in ADOPTERS.md and will be credited as founding adopters in the standards record.

### 8.2 Known Limitations

**Self-declared trust.** The most significant limitation is that `trust.level` is self-declared. A profile claiming `level: "enterprise"` without any genuine attestation is structurally valid and indistinguishable from a legitimately attested profile unless the consumer independently verifies `attestation.evidence_url`. ACPM's threat model (SECURITY.md) treats this explicitly: the standard specifies the shape of a trust claim, not a verification protocol, and consuming platforms that need higher assurance must do their own verification. `provenance.signature`, if present and verified by the consumer against a trusted public key, makes the profile tamper-evident — a profile with a valid signature cannot have been modified after signing — but does not itself verify the truth of the claims it covers.

**Point-in-time cost and SLA.** The `cost_profile` and `sla` blocks are point-in-time declarations with no refresh or TTL mechanism. A profile published six months ago may reflect a pricing tier that no longer exists. v1.0.0 does not include an `expires_at` or `last_verified_at` field on these blocks — a known gap noted in the SPECIFICATION's §G and §L. Consumers that rely on cost or SLA data for automated decisions should treat profile freshness as a function of `provenance.updated_at` and should re-validate against vendor documentation for decisions with material financial or operational consequences. A `cost_profile.valid_until` field is a candidate for a future MINOR version.

**Missing field: committed vs. observed SLA.** The `sla` block does not distinguish between contractual commitments and observed historical figures. A platform may honestly publish a 99.95% `uptime_pct` as an observed figure rather than a contractual guarantee. This distinction matters for SLA-based routing in regulated deployments. The gap is noted in SPECIFICATION §G; a `sla.type` enum (`committed`/`observed`) is a candidate for a future MINOR version.

### 8.3 The Three-Standard Stack

ACPM is the third layer of a three-standard stack that, taken together, describes the full "find it, trust it, know what it costs, run it" workflow for an AI agent:

- **AAIF (SC-006)** [5] — defines what an agent *is* and how to run it: goal, model routing, tools, memory, orchestration, guardrails, compliance requirements.
- **AREG (SC-013)** [12] — defines where to *find* it: registry listing, capability-based search, version and status filtering.
- **ACPM (SC-014)** — defines what it *offers* and under what terms: capability fidelity, trust posture, cost, SLA, delegation permissions.

None of the three requires the others — each is independently useful. But together they close the capability description gap that prevents automated, multi-vendor agent orchestration from operating on standard data rather than bespoke integrations.

---

## 9. Conclusion

Multi-agent AI systems today lack a standard vocabulary for describing what an agent, platform, tool, or model offers in machine-readable, vendor-neutral terms. Capability claims are informal, trust is unstructured, cost requires vendor-specific integration, and SLA data is inaccessible to the orchestration layer. The consequence is that the automated capability matching, trust gating, cost-optimised routing, and SLA-based dispatch that production multi-agent systems require are either implemented ad hoc — bespoke per-vendor parsers, hardcoded trust lists, manual pricing lookups — or not implemented at all.

ACPM (SC-014) defines a JSON Schema 2020-12 document format — a **capability profile** — that any AI subject can publish to declare what it supports, at what fidelity, under what trust and cost terms, and subject to what delegation constraints. The four support levels, five trust tiers, structured cost and SLA blocks, Condition-typed delegation rules, and compliance posture together provide the minimum vocabulary needed for automated orchestration decisions without bespoke per-vendor integration. The dot-namespaced capability vocabulary is deliberately aligned with AAIF's `required_capabilities[]` field, making requirement-to-offering matching a direct set operation. Five cumulative conformance levels from Basic to Enterprise make the standard immediately useful at any depth of adoption.

ACPM v1.0.0 is a Proposed Standard in Schema Commons, licensed CC BY 4.0. The schema and examples validate; there are no implementations and no adopters as of the publication date. The standard is a necessary pre-condition for an open, automated AI agent marketplace — a layer that the ecosystem does not yet have. Adoption by a registry or marketplace implementation would validate the schema design against real-world profiles and surface the open questions around trust verification freshness (a TTL mechanism for attestations), cost data staleness (a `valid_until` field for cost blocks), and the committed/observed SLA distinction — all noted as priority items for the next MINOR version.

---

## References

[1] Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels," BCP 14, RFC 2119, IETF, March 1997. https://www.rfc-editor.org/rfc/rfc2119

[2] Leiba, B., "Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words," BCP 14, RFC 8174, IETF, May 2017. https://www.rfc-editor.org/rfc/rfc8174

[3] Leach, P., Mealling, M., and R. Salz, "A Universally Unique IDentifier (UUID) URN Namespace," RFC 4122, IETF, July 2005. https://www.rfc-editor.org/rfc/rfc4122

[4] Wright, A., Andrews, H., Hutton, B., and G. Dennis, "JSON Schema: A Media Type for Describing JSON Documents," Internet-Draft draft-bhutton-json-schema-01, IETF, December 2020. https://json-schema.org/draft/2020-12

[5] van Bussel, B., "Autonomous Agent Interchange Format (AAIF)," Schema Commons SC-006, Observalytics SL, June 2026. https://github.com/Observalytics-SL/SC-006/

[6] Google LLC, "Agent2Agent (A2A) Protocol," 2025. https://a2aprotocol.ai

[7] Anthropic, "Model Context Protocol," 2024. https://modelcontextprotocol.io

[8] Anthropic / Model Context Protocol Contributors, "Model Context Protocol Development Roadmap," modelcontextprotocol.io/development/roadmap, March 5, 2026.

[9] Sun, Y., "HACP: A Capability-Contract Protocol for AI Agents and Edge Hardware," draft-sunyi-hacp-protocol-00, IETF Individual Submission, 2026.

[10] OWASP, "Agent Observability Standard (AOS) — Agent Bill of Materials (AgBOM)," OWASP / trustworthyagents, 2026. https://aos.owasp.org

[11] National Institute of Standards and Technology, "Artificial Intelligence Risk Management Framework (AI RMF 1.0)," NIST AI 100-1, U.S. Department of Commerce, January 2023. https://doi.org/10.6028/NIST.AI.100-1

[12] van Bussel, B., "Agent Registry (AREG)," Schema Commons SC-013, Observalytics SL, 2026. (forthcoming)

[13] Splunk, AWS, et al., "Open Cybersecurity Schema Framework (OCSF)," 2022. https://schema.ocsf.io

---

*Licensed CC BY 4.0. Part of [Schema Commons](https://github.com/Observalytics-SL). To cite this paper, see `CITATION.cff` in the repository.*
