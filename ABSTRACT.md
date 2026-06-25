# SC-014 ACPM — Abstract

**Standard:** SC-014 · **Full name:** Agent Capability and Profile Model
**Acronym:** ACPM · **Version:** 1.0.0 · **Status:** Proposed
**License:** CC BY 4.0 · **Date:** June 2026
**Repository:** https://github.com/Observalytics-SL/acpm
**Schema:** https://raw.githubusercontent.com/Observalytics-SL/acpm/main/schema/profile.schema.json
**Namespace:** https://github.com/Observalytics-SL/acpm/vocab#

---

## One-paragraph abstract (for directories, registries, and citations)

The Agent Capability and Profile Model (ACPM, SC-014) is an open, vendor-neutral JSON Schema standard for describing what an AI agent, platform, tool, or model actually offers, independent of how it is run or discovered. An ACPM document — a "capability profile" — declares a subject's identity, a dot-namespaced list of supported capabilities with a support level (native, emulated, partial, or none), the tool-invocation protocols, LLM models, and memory backends it exposes, its trust and attestation posture (including sandboxing), its pricing model and unit cost, its service-level commitments, structured delegation rules governing inbound and outbound work handoff, and its compliance posture (data residency, certifications, PII handling). ACPM profiles cross-reference SC-006 (AAIF) agent definitions and SC-013 (ARP) registry entries, and are designed to be consumed by registries, marketplaces, and orchestrators that need to compare capabilities, gate on trust, and reason about cost and SLA without bespoke per-vendor parsing. ACPM is part of Schema Commons, a family of open, CC BY 4.0 licensed data standards.

---

## Extended abstract (for conference submission, IETF I-D, or W3C Note)

### Problem

The AI agent ecosystem has converged on two adjacent but distinct problems: how to *run* a portable agent definition (SC-006 AAIF), and how to *discover* an agent in a registry (SC-013 ARP). Neither addresses a third, equally important problem: how to describe, compare, and trust what an agent, the platform running it, the tools it calls, or the underlying model actually offers. Today this information lives in marketing pages, ad hoc vendor JSON, and free-text documentation. There is no canonical way to ask whether a subject supports a given capability at what fidelity, what trust evidence backs that claim, what it costs, what SLA applies, or under what conditions it may delegate work to or accept work from another subject. Registries and orchestrators are forced to write bespoke per-vendor parsers or to skip capability/trust/cost comparison entirely.

### Contribution

ACPM defines a capability profile document as a JSON Schema (draft 2020-12) object with the following normative components:

1. **Subject identity** — `subject_type` (agent, platform, tool, or model), name, vendor, version, homepage, and `sc_refs[]` cross-references into other Schema Commons documents (an SC-006 agent_id, an SC-013 registry entry id).
2. **Capabilities** — a list of dot-namespaced capability ids (deliberately using the same vocabulary as SC-006 AAIF's `required_capabilities[]`) each with a `support` level (native / emulated / partial / none), an optional version constraint, and free-form notes.
3. **Tool, model, and memory inventories** — structured lists of supported tool-invocation protocols (function/MCP/HTTP/OpenAPI) with concurrency and auth-method limits, exposed LLM models with context window and modality data, and memory backends with their available scopes.
4. **Trust and attestation** — a five-tier trust level (untrusted → sandboxed → verified → attested → enterprise), an attestation block naming the issuer, method (self / third-party / formal audit), evidence URL, and validity window, and a sandbox block describing isolation strength and network egress policy.
5. **Cost and SLA** — a pricing model, currency, per-unit cost, and free-tier allowance; and an SLA block covering uptime, p50/p99 latency, support tier, and incident-response commitment.
6. **Delegation rules** — structured inbound/outbound rules naming an optional target (by profile id, capability id, or minimum trust level), an optional structured Condition (reusing SC-006 AAIF's Condition type verbatim), and an action (allow / deny / require_approval).
7. **Compliance and provenance** — data residency regions, free-form certifications, PII-handling mode, and authorship/licensing/signature metadata.

The minimum conformant document requires only `sc_standard: "SC-014"` and `subject` (with `subject_type` and `name`).

### Design rationale

ACPM follows the same design register as SC-006: declare rather than bind (a profile is a claim, not an enforced contract); composable (capability ids interoperate with AAIF's vocabulary by convention, not by coupling the schemas); honestly advisory where it cannot be enforced (`provenance.signature` and the entire trust block are claims a profile makes about itself — ACPM specifies the *shape* of the claim, not a verification mechanism); and strict by default (`additionalProperties: false` throughout, with the same `x-`/`_comment` extension escape hatch as AAIF).

### Relationship to adjacent standards

ACPM does not replace SC-006 (AAIF) or SC-013 (ARP). AAIF defines how an agent is *run*; ARP defines how an agent is *discovered* in a registry; ACPM defines what an agent, the platform running it, the tools it calls, or the underlying model *offers* — capability fidelity, trust, cost, and SLA. SC-013 registry entries may reference ACPM profiles to enrich a listing with comparable capability/cost/trust data; SC-006 agents declare `required_capabilities[]` using the same dot-namespaced vocabulary ACPM profiles use in `capabilities[].id`, so a profile can be checked against an agent's requirements directly.

ACPM also sits near several narrower, single-facet efforts outside Schema Commons: draft-sunyi-hacp-protocol-00 (HACP) declares per-session risk-tier capability contracts but only for physical edge hardware; MCP's own roadmap (modelcontextprotocol.io/development/roadmap) lists a still-forming "MCP Server Cards" effort for `.well-known` discovery metadata scoped specifically to MCP servers, which an ACPM `tool` profile for an MCP server should be designed to map onto; and OWASP's Agent Bill of Materials (AgBOM, via the Agent Observability Standard project, building on CycloneDX/SPDX/SWID) inventories what an agent is built from rather than what it offers at runtime. See SPECIFICATION.md §J for the full comparison; no existing effort combines capability, trust, cost, SLA, and delegation across agent/platform/tool/model the way ACPM does.

### Validation

The schema is implemented as JSON Schema 2020-12. All three included examples (an agent profile, a platform profile, and a model profile) validate against the schema using the Schema Commons reference validator (`tools/validate.py`).

### Status and next steps

ACPM 1.0.0 is a **Proposed** standard: the schema and examples are complete and validate, but there are no implementations, no adopters, and no self-certification tooling yet. Priority gaps before a stable release: (1) a reference validator/CLI comparable to AAIF's `aaif` package; (2) the conformance self-certification report schema and `/.well-known` publication convention (currently only specified in prose, see CONFORMANCE.md); (3) at least one independent adopter publishing a real profile. Community contributions are invited.

---

## Keywords

AI agents · capability profile · trust and attestation · cost comparison · SLA · interoperability · JSON Schema · Schema Commons · service catalog · capability negotiation

---

## Citation

```bibtex
@techreport{acpm-sc014-v1,
  title     = {Agent Capability and Profile Model (ACPM) — SC-014 v1.0.0},
  author    = {{Schema Commons Working Group}},
  year      = {2026},
  month     = {June},
  type      = {Proposed Standard},
  institution = {Schema Commons},
  url       = {https://github.com/Observalytics-SL/acpm},
  note      = {CC BY 4.0. Schema: profile.schema.json (sc_version 1.0.0)}
}
```

---

## Contact

- **Issues and proposals:** https://github.com/Observalytics-SL/Frameworks/issues
- **Working group:** See CONTRIBUTING.md in the repository
- **General enquiries:** hello@observalytics.com *(forthcoming)*
