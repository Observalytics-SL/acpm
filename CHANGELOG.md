# SC-014 ACPM — Changelog

All notable changes to the Agent Capability and Profile Model are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
Versioning follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

A **MAJOR** version increment means an incompatible schema change (field removed, type changed, enum value removed).
A **MINOR** version increment means a backwards-compatible addition (new optional field or enum value).
A **PATCH** version increment means clarifications, documentation fixes, or non-normative example changes.

---

## [1.0.0] — 2026-06-16

Initial public draft.

### Added

- **Root** — `sc_standard` (const `"SC-014"`), `sc_version` (semver, default `"1.0.0"`), `profile_id` (uuid).
- **subject** — `subject_type` (agent/platform/tool/model), `name`, `vendor`, `version`, `homepage`, `sc_refs[]` (cross-references to SC-006/SC-013/other Schema Commons documents).
- **capabilities[]** — `id` (dot-namespaced), `support` (native/emulated/partial/none), `version_constraint`, `notes`.
- **tools[]** — `protocol` (function/mcp/http/openapi), `max_concurrent_calls`, `auth_methods[]` (bearer/api_key/oauth2/aws_sigv4/vault).
- **models[]** — `provider`, `model_id`, `context_window`, `max_output_tokens`, `modalities[]` (text/image/audio/video/tool_use).
- **memory[]** — `backend` (in_memory/redis/postgres/sqlite/pinecone/weaviate/qdrant/chroma/custom), `scopes[]` (user/session/task/long_term).
- **trust** — `level` (untrusted/sandboxed/verified/attested/enterprise), `attestation` (issuer/method/evidence_url/issued_at/expires_at), `sandbox` (isolation/network_egress).
- **cost_profile** — `pricing_model` (free/usage_based/subscription/tiered/custom), `currency`, `unit_cost` (per/amount), `free_tier` (included_units/period).
- **sla** — `uptime_pct`, `p50_latency_ms`, `p99_latency_ms`, `support_tier`, `incident_response.severity1_minutes`.
- **delegation_rules[]** — `direction` (outbound/inbound), `target` (profile_id/capability_id/trust_level_min), `condition` (Condition type, reused from SC-006 AAIF §V), `action` (allow/deny/require_approval).
- **compliance** — `data_residency[]` (EU/US/UK/APAC/custom), `certifications[]`, `pii_handling` (none/redact/encrypt/forbidden).
- **provenance** — `authored_by`, `created_at`, `updated_at`, `license`, `signature`.
- **Extension namespace** — `^x-` patternProperties and root `_comment`, copied from SC-006 AAIF §W.
- **Conformance levels** — Basic → Tooled → Trusted → Priced → Enterprise (SPECIFICATION §I).
- Three examples: `invoice-chaser-profile.json` (agent), `langgraph-platform-profile.json` (platform), `gpt4o-model-profile.json` (model).
- `context.jsonld`, `draft-schemacommons-acpm-00.xml` (IETF Internet-Draft).

### Known gaps (tracked for future MINOR releases)

- No conformance self-certification report schema or `/.well-known` publication convention yet (see CONFORMANCE.md).
- No community extension registry for capability ids (unlike SC-006's `registries/`).
- No field distinguishing "committed" vs. "observed" SLA figures.
- No reference validator/CLI beyond the shared `tools/validate.py` schema check.
