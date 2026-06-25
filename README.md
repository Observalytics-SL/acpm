# SC-014 · ACPM — Agent Capability and Profile Model

[![Schema Commons Standard](assets/schema-commons-badge.svg)](https://github.com/Observalytics-SL) ![Status Proposed](https://img.shields.io/badge/status-proposed-orange) [![Cite](https://img.shields.io/badge/cite-CITATION.cff-blue)](CITATION.cff)

> **One spec sheet for what an agent, platform, tool, or model actually offers.** [SC-006 AAIF](https://github.com/Observalytics-SL/aaif) defines how to *run* a portable agent. SC-013 AREG *(forthcoming)* defines how to *discover* one in a registry. Neither tells you what it's actually capable of, how much you can trust it, what it costs, or what SLA it carries. ACPM is that spec sheet — "OCSF is to security events what ACPM is to AI agent capabilities."

## The problem

Every agent, platform, tool, and model vendor describes its capabilities differently — in marketing pages, free-text docs, or ad hoc JSON. There is no canonical, comparable, machine-readable way to ask "does this support MCP tool calls?", "what's its trust level?", "what does it cost per run?", or "can I delegate to it?" Registries and orchestrators end up writing bespoke per-vendor parsers, and capability matching, trust gating, and cost/SLA comparison are done by hand or not at all.

## The solution

A single, vendor-neutral capability profile that any agent, platform, tool, or model can publish:

```json
{
  "sc_standard": "SC-014",
  "sc_version": "1.0.0",
  "profile_id": "...",
  "subject": { "subject_type": "agent", "name": "...", "vendor": "...", "sc_refs": [...] },
  "capabilities": [
    { "id": "tool.mcp", "support": "native" },
    { "id": "memory.vector", "support": "emulated" }
  ],
  "tools": [...],
  "models": [...],
  "memory": [...],
  "trust": { "level": "verified", "attestation": {...}, "sandbox": {...} },
  "cost_profile": { "pricing_model": "usage_based", "unit_cost": { "per": "run", "amount": 0.04 } },
  "sla": { "uptime_pct": 99.5, "support_tier": "standard" },
  "delegation_rules": [...],
  "compliance": { "data_residency": ["EU"], "pii_handling": "redact" },
  "provenance": { "authored_by": "...", "license": "CC-BY-4.0" }
}
```

Capability ids use the same dot-namespaced vocabulary as [SC-006 AAIF](https://github.com/Observalytics-SL/aaif)'s `required_capabilities[]` (`tool.mcp`, `memory.vector`, `orchestration.parallel`, ...), so an AAIF agent's requirements and an ACPM profile's offerings can be matched directly. ACPM profiles are referenced from SC-013 AREG registry entries and SC-006 AAIF agents via `sc_refs[]`.

## Status & honest expectations

ACPM is a **brand-new, proposed standard with zero implementations and zero adopters**. It has not been used in production anywhere. The schema and examples validate cleanly today, but nothing yet generates, consumes, or enforces an ACPM profile in a real system. If you adopt it early, you are shaping the spec, not following an established one — say so, and we'll credit you as a founding adopter in [ADOPTERS.md](ADOPTERS.md).

ACPM does not replace SC-006 (AAIF) or SC-013 AREG; it sits alongside them as the capability/trust/cost layer that both can reference.

## An open standard, not just a schema

- **Cite it.** [`CITATION.cff`](CITATION.cff) + a placeholder Zenodo DOI; see [PUBLISHING.md](PUBLISHING.md) for the arXiv / IETF Internet-Draft / W3C Community Group paths.
- **Self-certify conformance.** Pick a level from the table below and check your profile against it; see [CONFORMANCE.md](CONFORMANCE.md). The self-certification report format is **planned, not yet shipped** — see CONFORMANCE.md for what exists today versus what's still on paper.
- **Report issues safely.** [SECURITY.md](SECURITY.md) defines the threat model (mostly: nothing here is cryptographically enforced) and disclosure process.
- **Extend without forking.** The `^x-` namespace and `_comment` root key (mirrored from SC-006) let tools attach metadata without breaking strict validation.

## Who benefits

| Role | Benefit |
|------|---------|
| **Registries & marketplaces** (e.g. SC-013 AREG) | List agents/platforms/tools/models with comparable, structured capability and cost data instead of free-text claims |
| **Orchestrators** | Match an AAIF agent's `required_capabilities[]` against ACPM profiles before dispatch; gate by `trust.level` |
| **Procurement / FinOps** | Compare `cost_profile` and `sla` across vendors on a like-for-like basis |
| **Security & compliance teams** | Check `trust.attestation`, `compliance.data_residency`, and `compliance.pii_handling` before approving a subject for use |
| **Platform builders** | Publish one profile instead of restating capabilities in docs, sales decks, and support tickets |

## Files

| File | Description |
|------|-------------|
| [SPECIFICATION.md](SPECIFICATION.md) | Full specification: terminology (RFC 2119), object model, field dictionary, trust/cost/SLA/delegation model, conformance levels, mapping to prior art, references |
| [ABSTRACT.md](ABSTRACT.md) | Structured abstract for registry submission, IETF I-D, and citation |
| [CONFORMANCE.md](CONFORMANCE.md) | Conformance levels and the (planned) self-certification report format |
| [PUBLISHING.md](PUBLISHING.md) | Paths to DOI, arXiv, IETF Internet-Draft, and W3C Community Group |
| [SECURITY.md](SECURITY.md) | Threat model and vulnerability disclosure policy |
| [CITATION.cff](CITATION.cff) | Citation metadata (GitHub "Cite this repository" + Zenodo DOI) |
| [CHANGELOG.md](CHANGELOG.md) | Version history |
| [ADOPTERS.md](ADOPTERS.md) | Self-service adopter registry |
| [schema/profile.schema.json](schema/profile.schema.json) | JSON Schema (draft 2020-12) — capability profile document (sc_version 1.0.0) |
| [context.jsonld](context.jsonld) | JSON-LD context mapping all ACPM terms to URIs (schema.org, Dublin Core, ACPM vocab) |
| [examples/invoice-chaser-profile.json](examples/invoice-chaser-profile.json) | Agent profile: the SC-006 "Invoice Chaser" agent, trust `verified`, usage-based cost |
| [examples/langgraph-platform-profile.json](examples/langgraph-platform-profile.json) | Platform profile: a LangGraph-style runtime, broad capabilities, trust `attested` |
| [examples/gpt4o-model-profile.json](examples/gpt4o-model-profile.json) | Model profile: GPT-4o, text+image+tool_use, usage-based per-1k-token cost |
| [draft-schemacommons-acpm-00.xml](draft-schemacommons-acpm-00.xml) | IETF Internet-Draft (xml2rfc v3) |

## Validate

```bash
python tools/validate.py
```

## Quick start: comparing two profiles

```python
import json, jsonschema

schema = json.load(open("schema/profile.schema.json"))

agent_profile = json.load(open("examples/invoice-chaser-profile.json"))
platform_profile = json.load(open("examples/langgraph-platform-profile.json"))

jsonschema.validate(agent_profile, schema)
jsonschema.validate(platform_profile, schema)

# Capability matching: does the platform support everything the agent needs?
needed = {c["id"] for c in agent_profile.get("capabilities", []) if c["support"] != "none"}
offered = {c["id"] for c in platform_profile.get("capabilities", []) if c["support"] in ("native", "emulated")}
missing = needed - offered
print("unsupported:", missing or "none — compatible")

# Trust gating
trust_order = ["untrusted", "sandboxed", "verified", "attested", "enterprise"]
ok = trust_order.index(platform_profile["trust"]["level"]) >= trust_order.index("verified")
print("platform meets minimum trust:", ok)
```

## Conformance levels

| Level | Requirements |
|-------|-------------|
| **Basic** | Valid schema; `subject` present |
| **Tooled** | Basic + `capabilities[]` non-empty |
| **Trusted** | Tooled + `trust.level` ∈ {verified, attested, enterprise} |
| **Priced** | Trusted + `cost_profile` present |
| **Enterprise** | Priced + `sla` + `compliance` + `provenance.signature` present |

See [SPECIFICATION.md §I](SPECIFICATION.md) for the full normative definitions.

## 📣 Ready-to-post LinkedIn announcement

> Comparing AI agents, platforms, and models today means reading marketing pages and hoping the claims are true.
>
> We just published **ACPM (SC-014)** — the Agent Capability and Profile Model. Think of it as "OCSF for AI agent capabilities": one canonical, machine-readable spec sheet for what an agent/platform/tool/model actually supports, how much you can trust it, what it costs, and what SLA backs it.
>
> ACPM profiles plug straight into **[AAIF (SC-006)](https://github.com/Observalytics-SL/aaif)** agents and **AREG (SC-013)** registry entries via cross-references, so orchestrators can do capability matching and trust gating without bespoke per-vendor parsing.
>
> It's brand new — zero adopters, zero implementations, fully open for first movers to help shape it.
>
> Part of **Schema Commons** — the Creative Commons for data schemas.
>
> #AIagents #OpenStandards #SchemaCommons #LLM #TrustAndSafety #MCP

## Companion standards

| Standard | What it adds |
|----------|-------------|
| [AAIF — SC-006](https://github.com/Observalytics-SL/aaif) | Agent definition: how to *describe and run* a portable agent |
| AREG — SC-013 *(forthcoming)* | Agent registry: how to *publish and discover* agents |

*Licensed CC BY 4.0 — part of [Schema Commons](https://github.com/Observalytics-SL).*
