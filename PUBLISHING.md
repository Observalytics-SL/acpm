# Publishing ACPM — paths to a recognised open standard

ACPM is designed to be published through several complementary channels, mirroring the path laid out for SC-006 AAIF. None are mutually exclusive; together they give the standard academic citability, standards-body legitimacy, and developer reach. This document is the roadmap from "a specification in a repository" to "a standard the field cites and implements" — written honestly for a standard that is, as of this writing, brand new.

## 0. Before submitting anywhere: prior-art status and submission readiness

**Submission status: READY FOR INITIAL FILING.** Prior-art diligence is complete (see §J of SPECIFICATION.md). The draft (draft-schemacommons-acpm-00.xml) is clean and submission-ready. Check for any published MCP Server Cards specification text and update §J before filing if the WG has since published concrete schema work.

This matters because of how IETF submission actually works: the instant a draft is filed with the IETF Datatracker, it is public and permanently archived. There is no private, embargoed, or draft-review submission step — "submitting" and "publishing to the world, forever, in a citable index" are the same action. Whatever prior-art diligence exists at the moment of submission is what reviewers, competitors, and future citers will see; it cannot be retroactively improved without a new, numbered revision sitting alongside the old one in the public record.

[SPECIFICATION.md §J ("Related Work & Mapping to Prior Art")](SPECIFICATION.md) reflects ACPM's current prior-art diligence: draft-sunyi-hacp-protocol-00 (HACP, a hardware-scoped capability-contract protocol), the Model Context Protocol's planned "MCP Server Cards" discovery-metadata effort, and OWASP's Agent Bill of Materials (AgBOM) concept. Two things are worth flagging before this draft is ever filed:

- **MCP Server Cards is still forming.** As of the cited roadmap snapshot (2026-03-05), the "Server Card WG" appears to have only just been chartered, with no concrete specification text yet. Re-check modelcontextprotocol.io/development/roadmap (and any successor specification it points to) for actual published Server Card text before ACPM finalizes any normative claim about how its `platform`/`tool` profile types are compatible with, or exportable as, an MCP Server Card. Treat the current §J language as a provisional, good-faith mapping, not a locked compatibility commitment.
- Re-run a fresh prior-art pass immediately before whichever submission path (IETF, arXiv, W3C CG) is used first — drafts and roadmaps move between when this diligence was done and when ACPM is actually filed.

## 1. Archival + DOI (citable today)

**Goal:** a permanent, versioned, citable artifact for academic and industrial reference.

- [`CITATION.cff`](CITATION.cff) makes the repository citable on GitHub and is consumed by Zenodo.
- Connect the repository to **Zenodo** and cut a tagged release → Zenodo mints a versioned **DOI**. Replace the placeholder DOI in `CITATION.cff`.
- The DOI is what papers, vendor docs, and the IETF draft reference. Each release gets its own DOI; a concept DOI always points at the latest.

## 2. Academic publication (the "PhD" track)

**Goal:** peer-reviewed grounding and a citable paper.

- ACPM's core contribution can be framed three ways: (a) the **capability comparability gap** across agent/platform/tool/model vendors as a measured problem; (b) ACPM as a **formal capability+trust+cost model** (a typed profile record plus a conformance lattice over levels, §I); (c) an **evaluation** — coverage analysis mapping existing ad hoc vendor capability disclosures (model cards, service catalogs, MCP server manifests) onto ACPM fields (§J).
- Strong venues: arXiv (cs.AI / cs.MA / cs.SE) for immediate availability, then a workshop/conference on agent infrastructure, trust & safety, or interoperability.
- Reproducibility: the schema and three examples in this repository are the seed artifact appendix.

## 3. IETF Internet-Draft (standards-track legitimacy)

**Goal:** review in an open standards body and a stable, referenceable document series.

- The specification already uses RFC 2119/8174 normative keywords (Conventions & terminology), has a **Security Considerations**-equivalent ([SECURITY.md](SECURITY.md)) and a **References** section.
- [`draft-schemacommons-acpm-00.xml`](draft-schemacommons-acpm-00.xml) is the xml2rfc v3 reformatting of `SPECIFICATION.md`. Submit to the IETF datatracker; seek a home in an AI/agents or applications-area discussion, or an Independent Submission via the ISE (Independent Submissions Editor) — same path as SC-006.

## 4. W3C Community Group (web + semantic-web reach)

**Goal:** multi-stakeholder governance and Linked-Data alignment.

- [`context.jsonld`](context.jsonld) maps ACPM terms to URIs (schema.org, Dublin Core, the ACPM vocab namespace). A **W3C Community Group** is a low-barrier home for cross-vendor participation — ACPM could plausibly share a CG with SC-006's "Portable AI Agents CG" given the overlapping audience, rather than spinning up a separate group immediately.

## 5. Reference runtime + adopters (running code)

**Goal:** "rough consensus and running code" — the credibility that matters most, and the area where ACPM is furthest behind SC-006.

- Unlike AAIF, ACPM currently has **no reference tooling at all** beyond the schema validator (`tools/validate.py`). The honest next steps, in priority order: (1) a small Python/TypeScript library that loads a profile and a needed-capability list and reports a match/gap, mirroring the Quick start snippet in [README.md](README.md); (2) a converter that derives a draft ACPM profile from an existing SC-006 agent definition or SC-013 registry entry; (3) the conformance report schema and test corpus described in [CONFORMANCE.md](CONFORMANCE.md).
- The **first independent adopter** to publish a real ACPM profile is recorded as a founding adopter in [ADOPTERS.md](ADOPTERS.md).

## Canonical URIs (to reserve)

| Purpose | URI |
|---------|-----|
| Standard landing page | `https://schemacommons.org/SC-014` |
| Profile schema `$id` | `https://schemacommons.org/SC-014/profile.schema.json` |
| Versioned schema | `https://schemacommons.org/SC-014/1.0.0/profile.schema.json` |
| JSON-LD context | `https://schemacommons.org/SC-014/context.jsonld` |
| Conformance report well-known *(planned)* | `/.well-known/acpm-conformance.json` |

When the domain is live, schema `$id`s SHOULD resolve to the schema file and SHOULD be content-addressable per version. Until then the `$id`s are stable identifiers, not necessarily dereferenceable URLs.

## Status

ACPM is a **Proposed** standard. It has no external adopters and no reference tooling beyond schema validation. The fastest credibility wins, in order: (1) mint a DOI, (2) ship the conformance report schema so level claims become machine-checkable, (3) land one independent adopter publishing a real profile, (4) post the arXiv write-up.
