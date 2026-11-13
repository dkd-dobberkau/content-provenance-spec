# Content Provenance Specification – Draft 0.2

November 13, 2026 · Olivier Dobberkau

## Status and changes since Draft 0.1

This is a working draft of an open, CMS-agnostic specification that lets automated agents verify who published a piece of web content, when, and on what sources. Draft 0.2 reconstructs Draft 0.1 (May 2026) and folds in the open points raised since.

| Change | Reason |
| --- | --- |
| Namespace is `https://cms-provenance.org/ns/v1` | Draft 0.1 used `contentprovenance.org`; the initiative's chosen domain is cms-provenance.org |
| Layer 2 now requires verification, not only presence, of signatures | The closed WordPress PR #294 was criticised for shallow cryptographic verification |
| Canonicalisation must be invisible-byte safe | The same PR injected invisible Unicode bytes that corrupted adjacent plugins |
| Section 5 adds a mapping to the MX vocabulary | Tom Cranstoun (The Gathering) proposed CPS as the CMS binding profile of MX |
| Terms that collide with MX are flagged | `lastSubstantiveRevision` vs `maintainedDate`, `conformanceLevel` vs `mxCompliance`, `generator` vs `provenanceOrigin` |
| Governance is an explicit open question | Vocabulary stewardship currently rests with one person; signing-identity governance is unresolved |

**Status.** Draft 0.2, November 2026. **Maintainers:** to be decided; initial drafting by dkd Internet Service GmbH, intended stewardship through a neutral body (TYPO3 Association, Open Website Alliance or a new working group). **Reference implementation:** TYPO3 first; Drupal, WordPress and Joomla are the stated follow-on goal.

## 1 Motivation

Schema.org describes what a page is; it does not say who made an assertion, when, or whether to trust it. As AI agents weight structured data more heavily for consequential decisions, that gap grows. This specification closes it with four layers that can be adopted independently or together.

The specification rewards content that is real, sourced and authored. It does not invent facts; it makes them legible when they exist.

## 2 Terminology

The keywords MUST, MUST NOT, SHOULD, SHOULD NOT and MAY are to be read as in RFC 2119.

| Term | Meaning |
| --- | --- |
| Document | A single addressable resource published by the CMS, typically one URL |
| Fact | A discrete claim inside a Document, addressable by URI fragment or annotation target |
| Provenance Record | Structured metadata on origin, authorship and history of a Document or Fact |
| Site Key | The key pair a publisher uses to sign Provenance Records |
| Audit Endpoint | A well-known URL on the publisher's domain exposing machine-readable Provenance Records |

## 3 The four layers

Each layer builds on the one before, but a site can stop at any layer and claim the matching conformance level (Section 4).

### 3.1 Layer 1: Provenance Record

Every published Document MUST carry a Provenance Record containing at least:

- A stable Document identifier that survives URL changes.
- One or more authors, each identified by a URI (an ORCID, a schema.org/Person profile URL, or an internal user URI exposed by the CMS).
- The first publication timestamp, immutable after first publication.
- The last *substantive* revision timestamp, distinct from any technical save or republish.
- Optionally a reviewer or approver, identified by URI.
- A source list; each source has a URL, a human-readable description and an access date.
- The CMS that produced the Record, by name and version.

Records MUST be exposed in the Document as JSON-LD using schema.org plus the namespace in Section 6. They MAY also be exposed as Microdata or RDFa.

### 3.2 Layer 2: Signature

A Provenance Record MAY be signed. If it is, the signature MUST cover a canonical hash of the Document content plus the Provenance Record.

- Hash: SHA-256.
- Signature: Ed25519 (recommended) or RSA-PSS with SHA-256.
- The public part of the Site Key MUST be discoverable through the Audit Endpoint and SHOULD also be linked from a key document at a stable URL via a `key` field.
- The signature MUST appear both in the Document (as a JSON-LD field) and in the Audit Endpoint entry.
- Rotation works through a `keyId`; historical public keys stay available at their stable URLs.
- **New in 0.2:** Verifiers MUST recompute the hash from the canonical form and check the signature against the published key. Reading the signature field is not verification.

The canonicalisation algorithm is defined in Annex A.

### 3.3 Layer 3: Fact-level addressing

A Document MAY expose individual Facts as separately addressable units. A Fact has:

- A URI of the form `https://example.org/document#fact-{id}`, or a Web Annotation target.
- A subject, predicate and object in a vocabulary the publisher declares.
- An optional source URI.
- An optional confidence or recency annotation.

Facts MUST be embedded in the Document and SHOULD also be exposed as Web Annotations (a W3C Recommendation) at the Audit Endpoint.

### 3.4 Layer 4: Audit Endpoint

Every implementing site MUST expose an Audit Endpoint at `/.well-known/content-provenance` returning JSON with:

- The site identity, including the public Site Key or a link to it.
- A list of published Documents, each with URL, stable identifier, hash, signature (if signed), publication timestamp and last substantive revision timestamp.
- Pagination through Link headers (RFC 8288).
- A `generator` field naming the CMS and version.

The Endpoint MAY expose Facts as Web Annotations under `/.well-known/content-provenance/facts`.

## 4 Conformance levels

A site conforms at one of four levels and claims it with a `conformanceLevel` field in the Audit Endpoint and in each Provenance Record.

| Level | Name | Requires |
| --- | --- | --- |
| L1 | Discoverable | Sitemap, clean HTML, canonical URLs, basic schema.org (roughly today's SEO baseline) |
| L2 | Provenance | Layer 1: every Document has a complete Provenance Record |
| L3 | Verifiable | Layers 1 and 2: Records are signed |
| L4 | Citable | Layers 1 to 4: Facts are addressable and the Audit Endpoint is live |

## 5 Relationship to existing standards

| Standard | Relationship |
| --- | --- |
| schema.org | Extended, not replaced. Records map to Article, CreativeWork, Person and, where applicable, Claim |
| C2PA (content credentials, originally for images) | Informs the signature layer; manifests are not required. A future profile MAY define manifest emission for text |
| W3C Web Annotations | Canonical external representation of Facts (Layer 3) |
| W3C DID and Verifiable Credentials | Site Keys MAY be expressed as DIDs; not required at this stage |
| Sitemaps protocol | Complemented, not replaced, by the Audit Endpoint |
| MX (Machine Experience vocabulary, The Gathering) | Proposed layering: MX governs the vocabulary, CPS defines how a CMS emits and serves it. Crosswalk below |

### 5.1 MX crosswalk: names to resolve first

These terms mean different things in CPS and MX, or the same thing under two names. They should be settled before any CMS field is fixed, to avoid later migrations.

| CPS | MX | Issue |
| --- | --- | --- |
| `lastSubstantiveRevision` | `maintainedDate` | Similar sounding, different semantics |
| `conformanceLevel` | `mxCompliance` | Format compliance vs editorial truthfulness must stay separate |
| `generator` | `provenanceOrigin` | Naming ambiguity; to be resolved before the Utrecht statement |

The crosswalk compared MX against the TYPO3 implementation described in the PRD, not against this specification text. A direct comparison against this document is still to be done.

## 6 Vocabulary

Fields that schema.org does not cover live in a small extension vocabulary under `https://cms-provenance.org/ns/v1`. The vocabulary file is normative and published alongside this specification. Who governs changes to it is an open question (Annex B).

## 7 Privacy and security considerations

- Authors MAY be identified pseudonymously through opaque URIs; real names are not required.
- The Audit Endpoint exposes publication metadata, which can be sensitive. Publishers MAY apply rate limits or authentication, provided good-faith auditors can still reach it.
- Private Site Key material MUST NOT appear in the Endpoint or in any Document.
- If a Site Key is compromised, the publisher MUST rotate it and SHOULD add a revocation entry to the Endpoint.
- **New in 0.2:** Whatever is embedded in a Document (identifiers, hashes, signatures) MUST NOT use invisible or non-printing characters, so that embedding cannot corrupt content handled by other plugins or extensions.
- **New in 0.2:** The trust anchor for a signature is the Site Key published by the site. Who is allowed to hold that key, and how a publisher proves who it is, is not defined here (Annex B).

## 8 Reference implementation

The TYPO3 reference implementation is the extension family [`provenance`](https://github.com/dkd-dobberkau/typo3-provenance) (Layer 1), with [`provenance_signing`](https://github.com/dkd-dobberkau/typo3-provenance-signing), [`provenance_facts`](https://github.com/dkd-dobberkau/typo3-provenance-facts) and [`provenance_audit`](https://github.com/dkd-dobberkau/typo3-provenance-audit) for Layers 2 to 4. Implementations for other CMS are encouraged. Conformance is determined by this specification, not by any implementation. Reviews of the implementation should be kept small enough to review; the data model for TYPO3 is kept in a separate document, the [TYPO3 PRD](https://github.com/dkd-dobberkau/cms-provenance.org/blob/main/docs/prd-typo3-extensions.md).

## 9 Versioning

This specification follows semantic versioning. A breaking change increments the major version. The Audit Endpoint declares the implemented version in a `specVersion` field.

## Annex A: Canonicalisation (to be defined in 0.3)

Working assumption: apply JCS (JSON Canonicalization Scheme, RFC 8785) to the JSON-LD Provenance Record, plus a canonical extraction of the Document text. The text pipeline serialises HTML deterministically, strips presentation markup, keeps structural semantics and normalises invisible characters. External review is planned, ideally from someone with W3C Verifiable Credentials and JCS experience.

## Annex B: Open questions

- The exact canonicalisation pipeline (Annex A).
- Reviewer signatures: co-signature on the same Record, or a separate Record type.
- Whether the Audit Endpoint stays a paginated listing or gains a query interface.
- How to express a deprecated Document (404, redirect or tombstone Record).
- Federation: whether one Endpoint can reference Records hosted elsewhere.
- Editor UX expectations for Layer 3.
- Signing-identity governance: who may hold a Site Key and how a publisher proves it.
- Vocabulary governance: how the namespace is stewarded beyond a single person, and how it relates to MX under The Gathering.
- Whether `generator` is renamed or aliased to `provenanceOrigin` (Section 5.1).

## Annex C: Non-goals

- The specification does not rank, score or evaluate content quality; it makes provenance legible.
- It does not prescribe a business model.
- It does not require any commercial registry. Public registries may emerge but are not part of conformance.
