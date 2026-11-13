# Content Provenance Specification

An open, CMS-agnostic specification that lets automated agents and human
auditors verify who published a piece of web content, when, and on what
sources.

**Current version: [Draft 0.3](specification.md)** (November 2026). Working
draft, not stable. Expect breaking changes before 1.0.

Draft 0.3 defines canonicalisation and signing normatively
([Annex A](specification.md#annex-a-canonicalisation-and-signing-normative)),
with test vectors and a reference implementation in the
[conformance repository](https://github.com/dkd-dobberkau/content-provenance-conformance).

## The four layers

| Layer | What it adds | Conformance level |
| --- | --- | --- |
| 1 Provenance Record | Authors, timestamps, sources as JSON-LD | L2 Provenance |
| 2 Signature | Ed25519 signature over a canonical hash | L3 Verifiable |
| 3 Fact-level addressing | Individually addressable claims | L4 Citable (with 4) |
| 4 Audit Endpoint | `/.well-known/content-provenance` | L4 Citable (with 3) |

Vocabulary namespace: [`https://cms-provenance.org/ns/v1`](https://cms-provenance.org/ns/v1)
(JSON-LD context: [`vocabulary/v1/context.jsonld`](vocabulary/v1/context.jsonld))

## Implementations

- **TYPO3** (reference, in development):
  [provenance](https://github.com/dkd-dobberkau/typo3-provenance),
  [provenance_signing](https://github.com/dkd-dobberkau/typo3-provenance-signing),
  [provenance_facts](https://github.com/dkd-dobberkau/typo3-provenance-facts),
  [provenance_audit](https://github.com/dkd-dobberkau/typo3-provenance-audit)
- Drupal, WordPress, Joomla: maintainers sought

Conformance test suite:
[content-provenance-conformance](https://github.com/dkd-dobberkau/content-provenance-conformance)

## Contributing

Open an issue for questions, objections and proposals, especially on the open
questions in Annex B of the specification. Contact:
[hello@cms-provenance.org](mailto:hello@cms-provenance.org). Project site:
<https://cms-provenance.org>.

## License

[CC-BY-4.0](LICENSE).
