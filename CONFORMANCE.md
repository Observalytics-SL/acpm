# ACPM Conformance & Self-Certification

> **How a profile (or a platform consuming profiles) proves it "speaks ACPM."** Conformance is per-level and self-certified — the same model SC-006 AAIF uses. There is no gatekeeper; credibility comes from the claim being checkable against this document and the schema. Normative level definitions are in [SPECIFICATION §I](SPECIFICATION.md).

## 1. Pick a level

**Levels** (cumulative — see [§I](SPECIFICATION.md)): `Basic` → `Tooled` → `Trusted` → `Priced` → `Enterprise`.

| Level | A conforming profile MUST satisfy… |
|-------|-------------------------------------|
| Basic | Validates against `schema/profile.schema.json`; `subject` present. |
| Tooled | Basic + `capabilities[]` non-empty. |
| Trusted | Tooled + `trust.level` is one of `verified`, `attested`, `enterprise`. |
| Priced | Trusted + `cost_profile` present. |
| Enterprise | Priced + `sla` present + `compliance` present + `provenance.signature` present. |

Most profiles should target **Tooled** or **Trusted** first. **Enterprise** requires committing to SLA and compliance disclosure and a signature field — don't claim it speculatively.

## 2. Validate today

The schema-level check that exists today:

```bash
python ../tools/validate.py SC-014-agent-capability-profile-model
```

This confirms the profile is well-formed JSON Schema 2020-12 conformant to `profile.schema.json`. It does **not** yet check the level-specific predicates in the table above — that check is currently manual (read the table, check your document against it).

## 3. What's planned but not yet shipped

Unlike SC-006, which has a working `schema/conformance-report.schema.json`, a `/.well-known/aaif-conformance.json` publication convention, and a `tools/test_conformance.py` corpus, **ACPM does not yet have any of that tooling**. This is being honest about the current state, not a hidden roadmap:

- **Planned:** `schema/conformance-report.schema.json` — a machine-readable report naming the claimed level, the subject's `profile_id`, and a test-suite result, publishable at a well-known path (candidate: `/.well-known/acpm-conformance.json`).
- **Planned:** a level-predicate test corpus (`tests/conformance/valid/` and `invalid/` fixtures) comparable to AAIF's, so level claims become machine-checkable rather than self-attested by reading the table above.
- **Not planned (by design):** any cryptographic verification of `trust.attestation` or `provenance.signature` as part of ACPM tooling itself — see [SECURITY.md](SECURITY.md). ACPM defines the *shape* of these claims; verifying them is a consuming platform's responsibility.

If you need the report format or test corpus now, open an issue — it is the most-requested next increment for this standard.

## 4. List yourself

Add a row to [ADOPTERS.md](ADOPTERS.md) describing which level you're claiming and for which profile(s). Listing is self-service and free; until the report format ships, the claim is verified by anyone re-reading your published profile against §I.

## What self-certification is and isn't

- **Is:** a public, reproducible statement that can be checked against the schema and the level table by hand or by script today, and against a machine-readable report once that tooling ships.
- **Isn't:** a guarantee that `trust.level`, `cost_profile`, or `sla` claims inside the profile are independently verified. Those are the subject's own claims (§F, §G of SPECIFICATION.md) — conformance to ACPM means the *document* is well-formed and satisfies the level's structural predicates, not that its content has been audited.

## Conformance changes between versions

A profile's `sc_version` names the ACPM schema version it targets. Every change is SemVer (see the root [GOVERNANCE.md](../GOVERNANCE.md)); a MINOR bump never invalidates an existing conformant profile.
