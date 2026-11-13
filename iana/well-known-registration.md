# Registration request: well-known URI `content-provenance`

Prepared for the IANA "Well-Known URIs" registry (RFC 8615). **Not filed
yet.** This file holds the request text and the steps, so that filing it is a
decision, not a writing task.

## Where and how

- Preferred: open an issue in
  [protocol-registries/well-known-uris](https://github.com/protocol-registries/well-known-uris)
  with the text below.
- Alternative: e-mail to the review list named in RFC 8615, subject
  `Request for well-known URI: content-provenance`.
- The registry policy is *Specification Required*: a designated expert
  reviews the request. The specification must be stably referenced, hence the
  tag `draft-0.3` in this repository.

## Before filing

1. **Engage a broader community first.** The registry repository recommends
   discussing a new well-known URI in a wider forum (for example IETF
   DISPATCH or a W3C Community Group) before requesting registration. A
   request backed by more than one implementer and a neutral steward is far
   more likely to be accepted without changes.
2. **Settle the change controller.** As long as stewardship sits with the
   initial authors, the maintainers and hello@cms-provenance.org are named.
   If a neutral body (TYPO3 Association, Open Website Alliance or a new
   working group) takes over first, name it instead.
3. **Point at a tagged version.** Use the `draft-0.3` URL below, or the tag of
   whatever version is current when filing.

## Issue text

> **Title:** Request for well-known URI: content-provenance
>
> **URI suffix:** content-provenance
>
> **Change controller:** Content Provenance Specification maintainers
> (contact: hello@cms-provenance.org). Change control is intended to move to
> a neutral steward as the specification matures.
>
> **Specification document(s):** Content Provenance Specification, Draft 0.3,
> Sections 3.4 and 10:
> https://github.com/dkd-dobberkau/content-provenance-spec/blob/draft-0.3/specification.md
>
> **Status:** provisional
>
> **Related information:** The Content Provenance Specification is an open,
> CMS-agnostic specification that lets automated agents and auditors verify
> who published a piece of web content, when, and on what sources. The
> well-known URI hosts the per-site Audit Endpoint: a paginated JSON listing
> of published documents with content hashes and Ed25519 signatures, the
> public signing keys (JWK) under `/keys`, and fact annotations (W3C Web
> Annotations) under `/facts`. A TYPO3 reference implementation exists
> (https://github.com/dkd-dobberkau/typo3-provenance-audit); test vectors and
> a verifier are published at
> https://github.com/dkd-dobberkau/content-provenance-conformance.
