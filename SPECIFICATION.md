# SC-014 · ACPM — Agent Capability and Profile Model — Specification

- **Standard:** SC-014 · **Acronym:** ACPM · **Version:** 1.0.0 (Proposed) · **License:** CC BY 4.0

---

## Conventions & terminology

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [BCP 14](https://www.rfc-editor.org/info/bcp14) ([RFC 2119](https://www.rfc-editor.org/rfc/rfc2119), [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174)) when, and only when, they appear in all capitals, as shown here.

| Term | Definition |
|------|------------|
| **ACPM document / profile** | A JSON instance conforming to `profile.schema.json`. |
| **Subject** | The agent, platform, tool, or model a profile describes. |
| **Capability** | A dot-namespaced runtime feature string a subject may offer, with a declared support level (§D, §C). |
| **Producer** | Software or a person that publishes an ACPM profile. |
| **Consumer** | Software that reads an ACPM profile to make a matching, trust, cost, or SLA decision. |
| **Conformance level** | One of the cumulative levels in §I that a profile satisfies. |
| **Condition** | The structured-or-string gating type reused verbatim from SC-006 AAIF, used in `delegation_rules[].condition` (§H). |

This document is normative. Examples and tables marked *informative* are not.

---

## A. Purpose & scope

ACPM defines a portable **capability profile**: a standalone description of what an AI agent, the platform that runs it, a tool it calls, or the underlying LLM actually offers — independent of how that subject is run (SC-006 AAIF) or discovered (SC-013 ARP). ACPM standardizes the *declaration* of capability fidelity, trust/attestation, cost, SLA, and delegation rules so that registries, marketplaces, and orchestrators can compare and match subjects without bespoke per-vendor parsing.

**In scope:** subject identity; capability support levels; tool/model/memory inventories; trust and attestation; cost structure; SLA commitments; delegation rules between subjects; compliance posture; provenance.

**Out of scope:** the execution loop of the subject (that is SC-006's domain); registry listing and discovery mechanics (that is SC-013's domain); cryptographic enforcement of any claim made in a profile (ACPM specifies the *shape* of trust/signature claims, not a verification protocol — see §F and SECURITY.md).

---

## B. Design principles

1. **Claims, not contracts** — a profile is what the subject *claims* to offer. ACPM does not itself verify claims; verification is the consuming platform's responsibility (§F).
2. **Vocabulary-compatible with AAIF** — `capabilities[].id` SHOULD use the same dot-namespaced strings as SC-006's `required_capabilities[]`, so an agent's needs and a profile's offerings compare directly without translation.
3. **Composable, not coupled** — ACPM cross-references SC-006 and SC-013 documents via `sc_refs[]` but does not import or extend their schemas. Each standard can evolve independently.
4. **Comparable by construction** — every offer-side concept (capability support, trust level, cost, SLA) uses a closed, ordered, or structured vocabulary rather than free text, so two profiles can be diffed or ranked mechanically.
5. **Strict by default, extensible at the edges** — `additionalProperties: false` everywhere, with a reserved `x-`/`_comment` escape hatch at the document root (§H-equivalent, copied from AAIF §W).
6. **Honest about enforceability** — fields like `provenance.signature` and `trust.attestation` are advisory unless a consuming platform independently verifies them. The spec says so explicitly rather than implying cryptographic guarantees it does not provide.

---

## C. Object model

```
CapabilityProfile (root)
├── sc_standard        "SC-014" (required)
├── sc_version         string (semver, default "1.0.0")
├── profile_id          (UUID)
└── subject              (required)
    ├── subject_type     agent | platform | tool | model
    ├── name / vendor / version / homepage
    └── sc_refs[]         ← cross-references to SC-006 / SC-013 / other SC docs
├── capabilities[]        ← id, support level, version_constraint, notes
├── tools[]                ← protocol, concurrency, auth_methods
├── models[]               ← provider, model_id, context_window, modalities
├── memory[]                ← backend, scopes
├── trust
│   ├── level             untrusted | sandboxed | verified | attested | enterprise
│   ├── attestation        ← issuer, method, evidence_url, issued_at, expires_at
│   └── sandbox             ← isolation, network_egress
├── cost_profile
│   ├── pricing_model      free | usage_based | subscription | tiered | custom
│   ├── currency / unit_cost / free_tier
├── sla
│   ├── uptime_pct / p50_latency_ms / p99_latency_ms
│   ├── support_tier / incident_response
├── delegation_rules[]     ← direction, target, condition, action
├── compliance
│   ├── data_residency[] / certifications[] / pii_handling
└── provenance
    ├── authored_by / created_at / updated_at / license / signature
```

---

## D. Field dictionary

### Root

| Field | Type | Req | Description |
|-------|------|-----|-------------|
| `sc_standard` | `"SC-014"` | ✓ | Standard identifier literal. |
| `sc_version` | string (semver) | | ACPM schema version. Default `"1.0.0"`. |
| `profile_id` | string (uuid) | | Globally unique profile identifier. SHOULD be stable across re-publication of the same profile. |

### subject

| Field | Type | Req | Description |
|-------|------|-----|-------------|
| `subject.subject_type` | enum | ✓ | `agent` / `platform` / `tool` / `model`. |
| `subject.name` | string | ✓ | Human-readable name of the subject. |
| `subject.vendor` | string | | Organisation or individual producing/operating the subject. |
| `subject.version` | string (semver) | | The subject's own version — independent of `sc_version`. |
| `subject.homepage` | string (uri) | | Canonical documentation or product URL. |
| `subject.sc_refs[]` | array | | `{standard, id}` — cross-references to other Schema Commons documents, e.g. `{"standard": "SC-006", "id": "<agent_id>"}` or `{"standard": "SC-013", "id": "<registry_entry_id>"}`. |

### capabilities[]

| Field | Type | Req | Description |
|-------|------|-----|-------------|
| `id` | string | ✓ | Dot-namespaced capability identifier, pattern `^[a-z][a-z0-9_]*(\.[a-z][a-z0-9_]*)*$`. SHOULD reuse SC-006 AAIF's vocabulary where the concept overlaps (`tool.mcp`, `memory.vector`, `orchestration.parallel`) and MAY extend it with new namespaces (`vision.*`, `audio.*`, `reasoning.*`, `auth.*`, `state.*`). |
| `support` | enum | ✓ | `native` (first-class) / `emulated` (achieved via workaround/composition) / `partial` (supported with material limitations) / `none` (explicitly absent — useful for comparison, not just omission). |
| `version_constraint` | string | | Semver range over `subject.version` for which this claim holds (e.g. `>=2.0.0`). |
| `notes` | string | | Free-form clarification, caveats, or doc links. |

### tools[]

| Field | Type | Description |
|-------|------|-------------|
| `protocol` | enum (required) | `function` / `mcp` / `http` / `openapi` — aligned with SC-006 AAIF `tools[].protocol`. |
| `max_concurrent_calls` | integer | Maximum concurrent tool calls the subject can issue or accept for this protocol. |
| `auth_methods` | array of enum | `bearer` / `api_key` / `oauth2` / `aws_sigv4` / `vault`. |

### models[]

| Field | Type | Description |
|-------|------|-------------|
| `provider` | string (required) | LLM provider identifier (free-form; SC-006 AAIF's `model.provider` vocabulary is RECOMMENDED). |
| `model_id` | string (required) | Model identifier in the provider's own notation. |
| `context_window` | integer | Maximum input context window, in tokens. |
| `max_output_tokens` | integer | Maximum output tokens per response. |
| `modalities` | array of enum | `text` / `image` / `audio` / `video` / `tool_use`. |

### memory[]

| Field | Type | Description |
|-------|------|-------------|
| `backend` | enum (required) | `in_memory` / `redis` / `postgres` / `sqlite` / `pinecone` / `weaviate` / `qdrant` / `chroma` / `custom` — aligned with SC-006 AAIF `memory[].backend`. |
| `scopes` | array of enum | `user` / `session` / `task` / `long_term` — aligned with SC-006 AAIF `memory[].scope`. |

### trust

| Field | Type | Description |
|-------|------|-------------|
| `level` | enum | `untrusted` / `sandboxed` / `verified` / `attested` / `enterprise` (ordered, increasing assurance — see §F). |
| `attestation.issuer` | string | Who issued the attestation (vendor itself, named auditor, certification body). |
| `attestation.method` | enum | `self` / `third_party` / `formal_audit`. |
| `attestation.evidence_url` | string (uri) | Link to the supporting report. |
| `attestation.issued_at` | string (date-time) | ISO 8601 UTC issuance timestamp. |
| `attestation.expires_at` | string (date-time) | ISO 8601 UTC expiry timestamp, if applicable. |
| `sandbox.isolation` | enum | `none` / `process` / `container` / `vm` — strongest isolation boundary the subject runs within. |
| `sandbox.network_egress` | enum | `none` / `allowlist` / `full`. |

### cost_profile

| Field | Type | Description |
|-------|------|-------------|
| `pricing_model` | enum | `free` / `usage_based` / `subscription` / `tiered` / `custom`. |
| `currency` | string | ISO 4217 code (e.g. `"USD"`). |
| `unit_cost.per` | enum | `request` / `token_1k` / `minute` / `run`. |
| `unit_cost.amount` | number | Price per unit, in `currency`. |
| `free_tier.included_units` | number | Free allotment per period. |
| `free_tier.period` | enum | `day` / `month`. |

### sla

| Field | Type | Description |
|-------|------|-------------|
| `uptime_pct` | number, 0–100 | Committed or observed uptime percentage. |
| `p50_latency_ms` | number | Median response latency. |
| `p99_latency_ms` | number | 99th-percentile response latency. |
| `support_tier` | enum | `community` / `standard` / `premium` / `enterprise`. |
| `incident_response.severity1_minutes` | integer | Committed Sev-1 response time, in minutes. |

### delegation_rules[]

| Field | Type | Req | Description |
|-------|------|-----|-------------|
| `direction` | enum | ✓ | `outbound` (this subject delegating out) / `inbound` (another subject delegating to this one). |
| `target.profile_id` | string (uuid) | | Restrict the rule to a specific counterparty profile. |
| `target.capability_id` | string | | Restrict the rule to delegation involving a specific capability. |
| `target.trust_level_min` | enum | | Minimum counterparty trust level for the rule to apply. |
| `condition` | Condition | | Additional structured (or string-hint) gate — see §H. |
| `action` | enum | ✓ | `allow` / `deny` / `require_approval`. |

At least one of `target.profile_id`, `target.capability_id`, or `target.trust_level_min` SHOULD be present per rule — this is non-normative authoring guidance, not a structural `oneOf` requirement, so that intentionally broad "allow/deny by direction alone" rules remain expressible.

### compliance

| Field | Type | Description |
|-------|------|-------------|
| `data_residency[]` | array of enum | `EU` / `US` / `UK` / `APAC` / `custom`. |
| `certifications[]` | array of string | Free-form (e.g. `"SOC2"`, `"ISO27001"`, `"HIPAA"`). |
| `pii_handling` | enum | `none` / `redact` / `encrypt` / `forbidden`. |

### provenance

| Field | Type | Description |
|-------|------|-------------|
| `authored_by` | string | Author identifier (username, email, or DID). |
| `created_at` | string (date-time) | ISO 8601 UTC creation timestamp. |
| `updated_at` | string (date-time) | ISO 8601 UTC last-modified timestamp. |
| `license` | string | SPDX ID or URL for this profile document. |
| `signature` | string | Optional base64-encoded signature over the canonical JSON. **Advisory only** — see §F and SECURITY.md. |

---

## E. Relationship to SC-006 and SC-013

ACPM does not replace either standard:

- **SC-006 (AAIF)** defines how an agent is *run*: identity, model routing, orchestration, tools, memory, runtime policy. An AAIF agent declares `agent.required_capabilities[]` — capabilities it *needs*. An ACPM profile declares `capabilities[]` — capabilities a subject *offers*. The two arrays SHOULD share the same dot-namespaced vocabulary so a consumer can compute `needed - offered` directly. ACPM profiles SHOULD reference the AAIF agent they describe via `subject.sc_refs[]` (`{"standard": "SC-006", "id": "<agent_id>"}`).
- **SC-013 (ARP)** defines how an agent is *discovered* in a registry. A registry entry's `required_capabilities` MAY point at an ACPM profile's `capabilities[].id` values, and a registry entry SHOULD link to the ACPM `profile_id` that describes the listed agent/platform, so a consumer can pull comparable capability/trust/cost/SLA data without the registry itself having to carry that data.
- Neither AAIF nor ARP requires ACPM. A document can validate against AAIF or ARP with no ACPM profile in existence; ACPM is additive context, not a dependency.

---

## F. Trust & attestation model

`trust.level` is an ordered, five-tier claim, not a cryptographic guarantee:

| Level | Meaning |
|-------|---------|
| `untrusted` | No claim has been verified by anyone. |
| `sandboxed` | The subject executes under some isolation boundary (see `trust.sandbox`), but capability/cost/SLA claims are unverified. |
| `verified` | Claims have been checked — by the publisher itself (self-certification) or informally — against observed behaviour. |
| `attested` | A named issuer has produced a dated, linkable attestation (`trust.attestation`) using a declared method. |
| `enterprise` | Attested, plus organisational accountability typically evidenced by `sla.support_tier` and `compliance` being populated. |

**Normative rules:**

- A profile MAY claim any `trust.level` without external verification. ACPM places no technical barrier in front of an inflated claim — see SECURITY.md for the resulting threat model.
- A consuming platform that gates on `trust.level` SHOULD additionally check that `trust.attestation.expires_at` (if present) has not passed before relying on an `attested` or `enterprise` claim.
- `provenance.signature`, if present, is a signature over the canonical JSON of the document. ACPM does **not** mandate a signature scheme, a key-distribution mechanism, or verification behaviour. A consuming platform MAY choose to verify it against an out-of-band public key it trusts; until it does, the signature is informational only.
- `trust.sandbox` describes the isolation the subject runs *under*, not the isolation a consumer gets when calling it remotely. A `container`-isolated subject called over an untrusted network still requires the consumer's own transport-level controls.

---

## G. Cost & SLA representation

`cost_profile` and `sla` are point-in-time claims, intended for comparison, not billing reconciliation:

- `cost_profile.unit_cost` names exactly one billing dimension (`per` + `amount`). Subjects with multi-dimensional pricing (e.g. per-request *and* per-token) SHOULD publish the dominant or most comparable dimension and use `notes`-equivalent free text in `subject.homepage`-linked docs for the rest, or publish multiple profiles per pricing tier.
- `cost_profile.free_tier` is additive: `included_units` per `period` are free before `unit_cost` applies.
- `sla.uptime_pct`, `p50_latency_ms`, and `p99_latency_ms` MAY be either contractual commitments or observed historical figures. Profiles SHOULD make clear in `provenance` or out-of-band documentation which is being claimed; ACPM does not currently have a field distinguishing "committed" from "observed" — this is a known gap, see §L.
- A consumer comparing two profiles' SLA or cost data SHOULD treat differing `currency` or `unit_cost.per` values as non-comparable without external conversion.

---

## H. Delegation rules and Condition expressions

`delegation_rules[]` governs whether the subject may hand work to (`outbound`) or accept work from (`inbound`) another subject. Each rule MAY scope itself to a specific counterparty (`target.profile_id`), a specific capability (`target.capability_id`), a minimum counterparty trust level (`target.trust_level_min`), and/or an additional structured `condition`.

The `condition` field reuses the **Condition** type defined in SC-006 AAIF SPECIFICATION §V, copied verbatim into this schema's `$defs.condition` with semantics unchanged:

- **Structured form (RECOMMENDED):** `{"lang": "jsonpath" | "jmespath" | "cel" | "jsonlogic" | "always" | "never", "expr": ..., "nl": "..."}`. `expr` is REQUIRED for `jsonpath`/`jmespath`/`cel`/`jsonlogic` and omitted for `always`/`never`. Deterministic and portable.
- **String form:** a bare string is accepted as a non-deterministic natural-language hint. MUST NOT be relied on for portable, reproducible gating.

**Normative rules** (identical to AAIF §V, restated for ACPM):

- A consumer that supports a declared `lang` MUST evaluate the structured form deterministically and MUST NOT fall back to an LLM interpretation of `nl`.
- A consumer that does not support a declared `lang` MUST treat the rule as unsatisfiable-unknown and apply a conservative default (deny for `outbound`, reject for `inbound`) rather than silently allowing.
- When `action` is `require_approval`, the consumer MUST obtain explicit approval before proceeding; treating `require_approval` as `allow` is a conformance violation.

---

## I. Conformance levels

| Level | Requirements |
|-------|-------------|
| **Basic** | Validates against `profile.schema.json`; `subject` present. |
| **Tooled** | Basic + `capabilities[]` non-empty. |
| **Trusted** | Tooled + `trust.level` ∈ {`verified`, `attested`, `enterprise`}. |
| **Priced** | Trusted + `cost_profile` present. |
| **Enterprise** | Priced + `sla` present + `compliance` present + `provenance.signature` present. |

Levels are cumulative. A profile claiming **Enterprise** satisfies every predicate of **Basic** through **Priced** as well. As with SC-006, conformance is **self-certified**: there is no central certifying authority. See [CONFORMANCE.md](CONFORMANCE.md) for what self-certification tooling exists today (the level predicates above) versus what is planned but not yet shipped (a machine-readable conformance report format, analogous to AAIF's `conformance-report.schema.json`).

---

## J. Related Work & Mapping to Prior Art

Before this draft is submitted anywhere, it is worth being explicit about which other efforts already occupy adjacent ground, and how ACPM relates to each — rather than letting the prior-art picture surface for the first time during external review.

The closest **domain-specific** precedent is **draft-sunyi-hacp-protocol-00** ("HACP: A Capability-Contract Protocol for AI Agents and Edge Hardware"), an active IETF individual draft. HACP is a JSON-RPC 2.0 protocol that lets an LLM agent discover, plan, execute, observe, and audit operations against *physical edge hardware* (GPIO, I2C, UART, sensors) through a security-checked daemon. Its capability model is close in spirit to ACPM's `capabilities[]` + `trust.level`: a server registers an immutable set of tools at startup via `tool.list`, each carrying a risk level from 0 to 3, and a session is capped at a configured maximum risk level — a session "MUST NOT execute a tool whose risk_level exceeds the session's configured maximum." That is genuinely the same underlying idea ACPM generalizes — "declare what this thing can do, and at what risk/trust tier, before you rely on it" — but HACP is narrowly scoped to hardware control, statically declared with no dynamic capability negotiation, and organized around risk tiers rather than ACPM's broader trust/cost/SLA/delegation combination. HACP is cited here as a narrow, complementary precedent for capability-contract thinking, not a competing general-purpose effort.

The closest **discovery-metadata** precedent is still forming rather than shipped: the Model Context Protocol's own roadmap (as published at modelcontextprotocol.io/development/roadmap, dated 2026-03-05) lists, under "Transport Evolution and Scalability," a planned **"MCP Server Cards"** effort — "a standard for exposing structured server metadata via a `.well-known` URL, so browsers, crawlers, and registries can discover a server's capabilities without connecting to it," attributed to a "Server Card WG" that the roadmap says is coordinating with "the broader industry AI-catalog effort." This is directly adjacent to ACPM's `subject_type: "platform"` / `subject_type: "tool"` profile concept: both are machine-readable, `.well-known`-style capability/metadata documents meant for discovery without a live connection. The honest, useful framing is not "ACPM vs. MCP Server Cards" but superset-and-instance: MCP Server Cards are scoped specifically to MCP servers, while ACPM is protocol-agnostic and covers agent/platform/tool/model profiles plus cost, SLA, trust, and delegation that a server card is not designed to express. An ACPM `tool`-type profile describing an MCP server **SHOULD** be designed so that its capability- and protocol-relevant fields are exportable as, or trivially mappable to, a future MCP Server Card, rather than requiring a separate, redundant description. Because the Server Card WG appears to have only just been chartered as of the cited roadmap snapshot, this mapping is necessarily provisional — see PUBLISHING.md §0 for the re-check this needs before ACPM finalizes any concrete compatibility claim.

The closest **supply-chain** precedent is the **Agent Bill of Materials (AgBOM)** concept referenced by OWASP's Agent Observability Standard (AOS) project (aos.owasp.org), which proposes inventorying what an agent or system is built from — models, tools, dependencies — using existing software-supply-chain formats such as CycloneDX, SPDX, and SWID. AgBOM overlaps with ACPM's `subject`/`tools[]`/`models[]` declarative inventory angle, but its center of gravity is different: AgBOM is about supply-chain provenance and inventory (what is *inside* this thing, for vulnerability and license tracking), while ACPM is about runtime capability, trust, cost, and SLA declaration (what this thing can *do*, and under what trust and cost terms it can be relied on). The two are complementary rather than competing; an ACPM profile's `subject.sc_refs[]` MAY point at a companion AgBOM document for the supply-chain detail ACPM itself does not try to capture.

Taken together, no found existing standard does what ACPM's core combination does: a single, vendor-neutral, cross-domain (agent/platform/tool/model) profile combining capability support level, trust/attestation, cost, SLA, and delegation rules in one document. The closest individual pieces — HACP's risk tiers, MCP Server Cards' discovery metadata, AgBOM's inventory model — each cover one facet of this narrowly and within a single domain. This section states that plainly rather than overselling: the *pieces* of ACPM's idea exist elsewhere; what is actually new is the combination, and the fact that it is meant to apply uniformly across agents, platforms, tools, and models rather than to one of those at a time.

**Comparison: domain-adjacent efforts (informative)**

| Effort | Scope | Overlap with ACPM | Relationship |
|--------|-------|--------------------|---------------|
| **HACP** (draft-sunyi-hacp-protocol-00) | JSON-RPC capability-contract protocol for LLM agents controlling physical edge hardware (GPIO/I2C/UART/sensors), with static per-session risk-level caps (0–3). | Capability declaration + trust/risk-tier gating, in spirit identical to ACPM's `capabilities[]` + `trust.level`. | Narrow, complementary precedent. Hardware-only; no cost, SLA, or delegation model. Not a competing general-purpose effort. |
| **MCP Server Cards** (planned, MCP roadmap, 2026-03-05) | `.well-known`-hosted structured metadata for MCP servers, for discovery without connecting. Owned by a newly forming Server Card WG. | Discovery-oriented capability/metadata document, same `.well-known` publication pattern as an ACPM `tool`/`platform` profile. | ACPM as superset / general case; Server Cards as the MCP-specific instance. An ACPM `tool` profile for an MCP server SHOULD be designed to be exportable as, or mappable to, a future Server Card. Still forming — re-check before finalizing compatibility claims (PUBLISHING.md §0). |
| **AgBOM** (OWASP Agent Observability Standard project, aos.owasp.org; builds on CycloneDX/SPDX/SWID) | Supply-chain inventory of what an agent/system is built from (models, tools, dependencies), for vulnerability and license tracking. | Declarative inventory overlap with ACPM's `subject`/`tools[]`/`models[]`. | Complementary, different center of gravity: AgBOM = what's inside (provenance), ACPM = what it can do and under what trust/cost/SLA terms (runtime offering). `sc_refs[]` MAY point at a companion AgBOM document. |

| ACPM concept | Maps to / inspired by |
|--------------|------------------------|
| Overall design intent | OCSF (Open Cybersecurity Schema Framework) — one canonical schema so disparate vendors describe the same kind of thing comparably; ACPM applies this idea to agent capabilities rather than security events. |
| `capabilities[]` with `support` levels | NIST capability maturity model style staged assessment (native/emulated/partial/none as a simplified maturity axis); OCI image manifest layers (declaring what is actually present, not just intended). |
| `tools[]`, `models[]` | Cloud provider "service catalog" schemas (AWS Service Catalog, Azure Marketplace listings) that enumerate what a service exposes and its limits. |
| `trust` block | A2A Agent Card's minimal capability/identity advertisement, extended with an explicit attestation and sandbox model that the Agent Card does not specify. |
| `tools[].protocol` negotiation | MCP server capability negotiation (the `initialize` handshake) — ACPM's `tools[]` is a static, publishable analogue of that runtime negotiation. |
| `delegation_rules[]` + Condition | SC-006 AAIF's Condition type (§V), reused verbatim; conceptually similar to OPA/Rego-style policy rules but scoped specifically to delegation between profiled subjects. |

---

## K. Versioning & changelog

| Version | Date | Change |
|---------|------|--------|
| 1.0.0 | 2026-06-16 | Initial public draft. Full field dictionary: subject, capabilities, tools, models, memory, trust, cost_profile, sla, delegation_rules, compliance, provenance. Conformance levels Basic→Tooled→Trusted→Priced→Enterprise. |

See [CHANGELOG.md](CHANGELOG.md) for the field-by-field breakdown of this initial release.

---

## L. FAQ

**Does ACPM replace AAIF or ARP?**
No. AAIF (SC-006) defines how to run an agent; ARP (SC-013) defines how to discover one in a registry. ACPM defines what a subject offers — capabilities, trust, cost, SLA, delegation. They compose; none of the three requires the others to be useful on its own.

**Can a profile lie about its `trust.level`?**
Yes, nothing in the schema prevents it. ACPM specifies the *shape* of a trust claim, not a verification mechanism. A consuming platform that needs assurance MUST do its own verification (checking `attestation.evidence_url`, requiring a signature it can verify, etc.). See SECURITY.md.

**Why is `provenance.signature` optional and unverified by the standard itself?**
Mandating a specific signature scheme (key format, distribution, revocation) is a separate, harder problem than describing capabilities. ACPM leaves room for a signature without prescribing how it is produced or checked, consistent with SC-006's same advisory treatment of `provenance.signature`.

**How do I describe a subject with multiple pricing tiers?**
Publish one profile per tier, or publish the dominant tier in `cost_profile` and document the rest out-of-band via `subject.homepage`. A future MINOR version may add a `cost_profile.tiers[]` array if this becomes a common need — see the gap noted in §G.

**Why isn't there a field for "committed vs. observed" SLA?**
This is a known gap (§G). v1.0.0 ships without it to keep the schema small; track or open an issue if you need it before the next MINOR release.

**What capability id should I use for a brand-new feature with no existing namespace?**
Pick the closest dot-namespace prefix (`tool.*`, `memory.*`, `orchestration.*`, `vision.*`, `audio.*`, `reasoning.*`, `auth.*`, `state.*`, or a new top-level prefix) and document it in `notes`. ACPM v1.0.0 does not yet have a community registry like AAIF's `registries/` (see CHANGELOG.md and the gaps noted throughout this document) — that is a candidate for a future MINOR version.

---

## M. References

### Normative references

- **[RFC 2119]** Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, 1997.
- **[RFC 8174]** Leiba, B., "Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words", BCP 14, RFC 8174, 2017.
- **[RFC 4122]** Leach, P., Mealling, M., and R. Salz, "A Universally Unique IDentifier (UUID) URN Namespace", RFC 4122, 2005.
- **[JSON-SCHEMA]** Wright, A., et al., "JSON Schema: A Media Type for Describing JSON Documents", Draft 2020-12.

### Informative references

- **[SC-006]** Schema Commons, "Autonomous Agent Interchange Format — SPECIFICATION.md", https://github.com/Observalytics-SL/SC-006/.
- **[SC-013]** Schema Commons, "Agent Registry Protocol — README.md", https://github.com/Observalytics-SL/SC-013/.
- **[MCP]** Anthropic, "Model Context Protocol", 2024, https://modelcontextprotocol.io.
- **[A2A]** "Agent2Agent Protocol", https://a2aprotocol.ai (or successor specification location).
- **[SPDX]** Linux Foundation, "SPDX License List", https://spdx.org/licenses/.
- **[HACP]** Sun, Y., "HACP: A Capability-Contract Protocol for AI Agents and Edge Hardware", draft-sunyi-hacp-protocol-00 (Work in Progress), IETF Individual Submission, 2026.
- **[MCP-ROADMAP]** Anthropic / Model Context Protocol contributors, "Development Roadmap", 2026-03-05, https://modelcontextprotocol.io/development/roadmap.
- **[OWASP-AOS]** OWASP, "Agent Observability Standard (AOS) project — Agent Bill of Materials (AgBOM)", https://aos.owasp.org. AgBOM builds on existing software-supply-chain inventory formats (CycloneDX, SPDX, SWID).

### Author / editor

Edited under the Schema Commons governance model. Contributions are licensed CC BY 4.0 (specification) and Apache 2.0 (tooling, where applicable). To cite, see [CITATION.cff](CITATION.cff).

---
