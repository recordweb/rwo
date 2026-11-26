# RecordWeb Ontology (RWO)

> Copyright © 2026 the Contributors to the RecordWeb Ontology Specification,
> published by the W3C RecordWeb Community Group under the
> [W3C Community Contributor License Agreement (CLA)](https://www.w3.org/community/about/process/cla-deed/).
> A human-readable summary is available.

## What is this?

RWO defines the RDF/OWL vocabulary used for semantic interoperability of the
[RecordWeb Protocol (RWP)](https://recordweb.github.io/rwp/) and the
[RecordWeb Concept (RWC)](https://recordweb.github.io/rwc/).

It provides the stable namespace:

```
https://w3id.org/recordweb/ontology#
```

used with the conventional prefix:

```turtle
@prefix rw: <https://w3id.org/recordweb/ontology#> .
```

The central class `rw:Record` is defined as a specialization of
[`prov:Entity`](https://www.w3.org/ns/prov#Entity) from the
[PROV-O](https://www.w3.org/TR/prov-o/) ontology, so that Records can be
represented and processed in any PROV-O-compatible tooling while remaining
the primary term of the RWP specification.

## Relationship to RWP and RWC

- **RWC** (RecordWeb Concept) — informative Community Group Report,
  describes the model and rationale.
- **RWP** (RecordWeb Protocol) — normative specification, defines the
  protocol requirements (MUST/SHOULD/MAY).
- **RWO** (RecordWeb Ontology, this repository) — the RDF/OWL vocabulary
  that gives RWP/RWC terms a stable, dereferenceable, machine-readable
  identity for semantic-web and linked-data use.

RWO is versioned independently from RWP. RWP can remain stable while RWO
adds new mappings, annotations, or properties, provided existing term IRIs
are never removed or redefined.

## Files

- `ontology.ttl` — the ontology in Turtle (canonical source).
- `index.html` — human-readable documentation, served at
  `https://w3id.org/recordweb/ontology`.

## Namespace registration

This namespace is registered with [w3id.org](https://w3id.org/), the
permanent identifier redirection service, via a pull request to
[perma-id/w3id.org](https://github.com/perma-id/w3id.org). See that
repository's `recordweb/` entry for the redirect rules.

## License

This repository is published under the
[W3C Software and Document License](./LICENSE).

## Governance

Governance, contribution process, and Code of Conduct for RecordWeb are
defined centrally in
[recordweb/.github](https://github.com/recordweb/.github).
