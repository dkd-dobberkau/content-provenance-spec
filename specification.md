# Content Provenance Specification – Draft 0.3

November 13, 2026 · Olivier Dobberkau

## Status and changes

This is a working draft of an open, CMS-agnostic specification that lets automated agents verify who published a piece of web content, when, and on what sources.

### Changes in Draft 0.3

| Change | Reason |
| --- | --- |
| Annex A defines canonicalisation and signing normatively | Without a fixed algorithm, signatures cannot be checked by third parties |
| The content hash is carried in the Record as `contentHash`; the signature covers the canonical Record | One signature then binds text and Record, and the Record stays self-describing |
| Records may contain integers only, no fractional numbers | Removes the hardest part of JCS number serialisation and a common source of mismatches |
| Test vectors and a reference implementation are published in the conformance repository | Implementations can prove they compute the same bytes |
| Section 3.4 lists the defined sub-paths of the Audit Endpoint; Section 10 adds the IANA registration of the well-known URI suffix | RFC 8615 asks for well-known names to be registered, and implementations need a fixed path layout |

### Changes in Draft 0.2

Draft 0.2 reconstructed Draft 0.1 (May 2026) and folded in the open points raised since.

| Change | Reason |
| --- | --- |
| Namespace is `https://cms-provenance.org/ns/v1` | Draft 0.1 used `contentprovenance.org`; the initiative's chosen domain is cms-provenance.org |
| Layer 2 now requires verification, not only presence, of signatures | The closed WordPress PR #294 was criticised for shallow cryptographic verification |
| Canonicalisation must be invisible-byte safe | The same PR injected invisible Unicode bytes that corrupted adjacent plugins |
| Section 5 adds a mapping to the MX vocabulary | Tom Cranstoun (The Gathering) proposed CPS as the CMS binding profile of MX |
| Terms that collided with MX are resolved | `maintainedDate` adopted alongside `lastSubstantiveRevision`; `conformanceLevel` renamed to `cpsConformance`; `generator` kept, `provenanceOrigin` documented as the MX counterpart (Section 5.1) |
| Governance is an explicit open question | Vocabulary stewardship currently rests with one person; signing-identity governance is unresolved |

**Status.** Draft 0.3, November 2026. **Maintainers:** to be decided; initial drafting by dkd Internet Service GmbH, intended stewardship through a neutral body (TYPO3 Association, Open Website Alliance or a new working group). **Reference implementation:** TYPO3 first; Drupal, WordPress and Joomla are the stated follow-on goal.

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
- The last *substantive* revision timestamp (`lastSubstantiveRevision`), distinct from any technical save or republish.
- Optionally the date the content was last confirmed as still accurate (`maintainedDate`, adopted from MX). A confirmation without an edit updates `maintainedDate` only, never `lastSubstantiveRevision`.
- Optionally a reviewer or approver, identified by URI.
- A source list; each source has a URL, a human-readable description and an access date.
- The CMS that produced the Record, by name and version.

Records MUST be exposed in the Document as JSON-LD using schema.org plus the namespace in Section 6. They MAY also be exposed as Microdata or RDFa.

### 3.2 Layer 2: Signature

A Provenance Record MAY be signed. If it is, the signature MUST cover a canonical hash of the Document content plus the Provenance Record.

- Hash: SHA-256 over the canonical text of the Document, carried in the Record as `contentHash` (Annex A.3).
- Signature: Ed25519 (recommended) or RSA-PSS with SHA-256, over the canonical Record (Annex A.4, A.5). Verifiers MUST support Ed25519.
- The public part of the Site Key MUST be discoverable through the Audit Endpoint and SHOULD also be linked from a key document at a stable URL via a `key` field.
- The signature MUST appear both in the Document (as a JSON-LD field) and in the Audit Endpoint entry.
- Rotation works through a `keyId`; historical public keys stay available at their stable URLs.
- **New in 0.2:** Verifiers MUST recompute the hash from the canonical form and check the signature against the published key. Reading the signature field is not verification.

Canonicalisation, signing and verification are defined in Annex A.

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

The following paths below `/.well-known/content-provenance` are defined. Other sub-paths are reserved for future versions of this specification; implementations MUST NOT define their own.

| Path | Content |
| --- | --- |
| `/.well-known/content-provenance` | The Audit Endpoint listing; further pages through Link headers |
| `/.well-known/content-provenance/keys` | All Site Keys, including retired ones, as JWK (RFC 7517, RFC 8037) |
| `/.well-known/content-provenance/keys/{keyId}` | One Site Key; the target of a Record's `key` |
| `/.well-known/content-provenance/facts/{factId}` | One Fact as a Web Annotation (Layer 3) |

All of these resources MUST be readable without authentication by the auditors the publisher admits (Section 7) and SHOULD allow cross-origin reads (`Access-Control-Allow-Origin: *`), so that browser-based verifiers work.

## 4 Conformance levels

A site conforms at one of four levels and claims it with a `cpsConformance` field in the Audit Endpoint and in each Provenance Record.

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

These terms meant different things in CPS and MX, or the same thing under two names. They are settled in Draft 0.2 so that CMS fields can be fixed without later migrations.

| CPS | MX | Issue | Resolution |
| --- | --- | --- | --- |
| `lastSubstantiveRevision` | `maintainedDate` | Similar sounding, different semantics: last real edit vs last confirmation that the content is still accurate | Both kept. CPS adopts `maintainedDate` from MX as an optional Layer 1 field; merging them would lose information |
| `cpsConformance` (formerly `conformanceLevel`) | `mxCompliance` | Format compliance vs editorial truthfulness must stay separate | Renamed with a `cps` prefix so the two can never be confused |
| `generator` | `provenanceOrigin` | Naming ambiguity | `generator` kept, following HTML `meta name="generator"` and Atom. `provenanceOrigin` is the MX counterpart; consumers MAY treat them as equivalent |

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

## 10 IANA considerations

This specification registers the following entry in the "Well-Known URIs" registry established by RFC 8615:

| Field | Value |
| --- | --- |
| URI suffix | `content-provenance` |
| Change controller | Content Provenance Specification maintainers, contact <hello@cms-provenance.org>. Change control is intended to move to the neutral body that takes over stewardship (see Status). |
| Specification document(s) | This document, Section 3.4 and Section 10 |
| Status | provisional |
| Related information | Sub-paths are defined in Section 3.4. The suffix is used by the Audit Endpoint, the Site Key documents and Fact annotations of the Content Provenance Specification. |

The status is provisional while the specification is a draft. It will be changed to permanent with version 1.0.

## Annex A: Canonicalisation and signing (normative)

This annex turns a published Document and its Provenance Record into exactly defined bytes, so that any party can recompute the hash and check the signature. A reference implementation and test vectors are published in the [conformance repository](https://github.com/dkd-dobberkau/content-provenance-conformance) (`reference/python`, `vectors/canonicalization`). Where they disagree with this annex, this annex wins.

Overview:

1. Extract the canonical text of the Document (A.2) and hash it into `contentHash` (A.3).
2. Serialise the Record, including `contentHash` and excluding `signature`, with JCS (A.4).
3. Sign those bytes; add the `signature` object to the Record (A.5).

### A.1 Input

The input is the HTML of the Document as served at its URL, parsed with the HTML parsing algorithm of the WHATWG HTML Living Standard. Element names are compared by local name, regardless of namespace (SVG and MathML elements live in their own namespaces after parsing). Text is a sequence of Unicode code points.

### A.2 Canonical text

#### A.2.1 Content roots

The content roots are determined as follows, in this order:

1. If any element carries the attribute `data-cps-content`, the content roots are those elements in document order, except elements that are descendants of another such element.
2. Otherwise, the first `main` element.
3. Otherwise, the `body` element.

Publishers SHOULD mark the editorial content explicitly with `data-cps-content` or keep navigation, headers and footers outside `main`. Text outside the content roots is not covered by the signature.

#### A.2.2 Excluded subtrees

A walk over a content root skips an element and all its descendants if

- its local name is one of `head`, `script`, `style`, `template`, `noscript`, `iframe`, `object`, `embed`, `canvas`, `svg`, `button`, `input`, `select`, `textarea`; or
- it carries the attribute `hidden`; or
- it carries the attribute `aria-hidden` with the value `true` (ASCII case-insensitive); or
- it carries the attribute `data-cps-ignore`.

Comments and processing instructions are skipped. Publishers SHOULD mark per-request content inside a content root, such as form tokens or personalised greetings, with `data-cps-ignore`.

#### A.2.3 Text extraction

Walk each content root depth-first in document order and build a string:

- Before each content root and after it, emit a line break (U+000A).
- When entering and when leaving an element whose local name is one of `address`, `article`, `aside`, `blockquote`, `caption`, `dd`, `details`, `dialog`, `div`, `dl`, `dt`, `fieldset`, `figcaption`, `figure`, `footer`, `form`, `h1` to `h6`, `header`, `hgroup`, `hr`, `li`, `main`, `nav`, `ol`, `p`, `pre`, `section`, `summary`, `table`, `tbody`, `td`, `tfoot`, `th`, `thead`, `tr`, `ul`, emit a line break.
- For a `br` element, emit a line break.
- For an `img` element, emit the value of its `alt` attribute, if any.
- For a text node, emit its data. Outside a `pre` element, each U+000A in the data is emitted as U+0020; inside a `pre` element it is emitted unchanged.

Line breaks therefore come from document structure only; how the HTML source is wrapped has no effect.

#### A.2.4 Normalisation

Apply these steps to the extracted string, in order:

1. **Remove invisible and control characters.** Remove every code point in the following ranges. They are the code points of General Category Cf as of Unicode 15.1, frozen here so that the result does not depend on an implementation's Unicode version, plus the C0 and C1 controls not handled in step 3:
   U+0000–U+0008, U+000E–U+001F, U+007F–U+009F, U+00AD, U+0600–U+0605, U+061C, U+06DD, U+070F, U+0890–U+0891, U+08E2, U+180E, U+200B–U+200F, U+202A–U+202E, U+2060–U+2064, U+2066–U+206F, U+FEFF, U+FFF9–U+FFFB, U+110BD, U+110CD, U+13430–U+1343F, U+1BCA0–U+1BCA3, U+1D173–U+1D17A, U+E0001, U+E0020–U+E007F.
2. **Compose.** Convert to Unicode Normalization Form C.
3. **Unify whitespace.** Replace each of U+0009, U+000B, U+000C, U+000D, U+00A0, U+1680, U+2000–U+200A, U+2028, U+2029, U+202F, U+205F, U+3000 with U+0020.
4. **Collapse.** Split the string at U+000A into lines. In each line, replace every run of U+0020 with a single U+0020 and remove leading and trailing U+0020. Drop empty lines. Join the remaining lines with U+000A. There is no line break at the start or end.

The result is the **canonical text**. Encoded as UTF-8, without byte order mark, it forms the **canonical text bytes**.

Step 1 makes the hash insensitive to invisible bytes injected into content, as required in Section 7.

### A.3 Content hash

`contentHash` is the string `sha256:` followed by the SHA-256 digest of the canonical text bytes, written as 64 lowercase hexadecimal digits. It is a top-level property of the Record.

### A.4 Canonical Record

The **Record** is the JSON-LD object of the Document whose `@context` contains `https://cms-provenance.org/ns/v1`, either as the string value of `@context` or as an element of its array. If a Document contains more than one such object, it is not conformant at L3.

The **canonical Record bytes** are the UTF-8 encoding of the JSON Canonicalization Scheme (RFC 8785) serialisation of the Record after removing its top-level `signature` member. All other members, including `@context` and `contentHash`, are part of the canonical Record.

Records MUST NOT contain numbers other than integers in the range −(2^53−1) to 2^53−1. With that restriction, RFC 8785 reduces to: object members sorted by the UTF-16 code units of their names; no insignificant whitespace; strings escaped only for `"`, `\`, and code points below U+0020 (`\b`, `\f`, `\n`, `\r`, `\t`, otherwise `\u00xx` in lowercase hex); integers in plain decimal notation.

### A.5 Signature

#### A.5.1 Signing

1. Compute `contentHash` (A.3) and set it on the Record.
2. Compute the canonical Record bytes (A.4).
3. Sign those bytes with the private Site Key.
4. Add a top-level member `signature` to the Record:

```json
"signature": {
  "algorithm": "Ed25519",
  "keyId": "site-key-2026-11",
  "signatureValue": "base64url, no padding"
}
```

| `algorithm` | Signing operation |
| --- | --- |
| `Ed25519` | Ed25519 as in RFC 8032 (PureEdDSA) over the canonical Record bytes |
| `RSA-PSS-SHA256` | RSASSA-PSS as in RFC 8017 with SHA-256, MGF1 with SHA-256 and a 32-byte salt, over the canonical Record bytes; keys of at least 3072 bits |

`signatureValue` is the signature encoded in base64url without padding (RFC 4648, Section 5).

#### A.5.2 Verification

A verifier MUST perform all of the following steps. The signature is valid only if every step succeeds.

1. Fetch the Document and find the Record (A.4).
2. Compute the canonical text of the fetched Document (A.2) and its content hash (A.3). Compare it with the Record's `contentHash`. They MUST be equal.
3. Resolve the public key for `signature.keyId` through the Audit Endpoint (Section 3.4) or the `key` document. Retired keys remain valid for signatures made before their retirement.
4. Compute the canonical Record bytes (A.4) and verify `signature.signatureValue` against them with that key and `signature.algorithm`.

A Record that carries a `signature` but fails any step MUST be treated as unsigned, and the Document MUST NOT be reported as conforming at L3 or above.

### A.6 Test vectors

The conformance repository holds vectors under `vectors/canonicalization`. Each vector has an `input.html`, a hand-written `expected-text.txt` with the canonical text, a `record.json`, and an `expected.json` with `contentHash`, the canonical Record, its SHA-256 and the Record signed with the Ed25519 test key from RFC 8032, Section 7.1, TEST 1 (`vectors/keys/test-key.json`). An implementation conforms to this annex if it reproduces every `expected-text.txt` and every value in `expected.json`.

## Annex B: Open questions

- Whether Annex A should cover documents other than HTML (PDF, feeds).
- Reviewer signatures: co-signature on the same Record, or a separate Record type.
- Whether the Audit Endpoint stays a paginated listing or gains a query interface.
- How to express a deprecated Document (404, redirect or tombstone Record).
- Federation: whether one Endpoint can reference Records hosted elsewhere.
- Editor UX expectations for Layer 3.
- Signing-identity governance: who may hold a Site Key and how a publisher proves it.
- Vocabulary governance: how the namespace is stewarded beyond a single person, and how it relates to MX under The Gathering.

## Annex C: Non-goals

- The specification does not rank, score or evaluate content quality; it makes provenance legible.
- It does not prescribe a business model.
- It does not require any commercial registry. Public registries may emerge but are not part of conformance.
