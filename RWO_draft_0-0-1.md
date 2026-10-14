# RecordWeb Operations (RWO)

Draft Community Group Report, working draft 0.0.1 (skeleton)

Status: **Draft for discussion.** Normative text is provisional. Open points are marked as `Issue`.

---

## Abstract

The RecordWeb Operations specification (RWO) defines the basic operations that a custodian offers to producers and consumers of Records in the sense of the RecordWeb Protocol [RWP]: **Retrieve**, **Submit**, **Erase** and **Describe**. It defines the operations abstractly, independent of any transport, and defines a first HTTP binding. Its purpose is interoperability: when a namespace moves from one custodian to another, the same operations MUST remain available with the same meaning.

---

## 1. Introduction and Scope

### 1.1 Purpose

RWP defines the roles producer, custodian and consumer and the structure of a Record. It does not define how a producer hands a Record to a custodian, how a Record is updated or erased, or how a consumer asks a custodian for it. RWO closes this gap with a minimal, stable set of operations.

### 1.2 Scope

RWO defines:

- the abstract operations Retrieve, Submit, Erase and Describe, including their options and default behaviour;
- how options narrow an operation using the structure of the Record DID;
- the capability description published by a custodian;
- error types;
- an HTTP binding.

### 1.3 Out of scope

The following are deliberately out of scope of RWO:

- **Handover of responsibility.** Retrieve or Submit alone MUST NOT be interpreted as a handover. Responsibility changes only when the endpoint registered in the DID document changes.
- **Namespace migration process.** Which Records are transferred, in which order, and when the endpoint is switched is organisational and operational.
- **Authentication, authorisation and access policy.**
- **Legal basis for erasure.** RWO defines that Erase exists, not when it is lawful.
- **Search and discovery across namespaces.**
- **Proof of handover.**

> Issue: Confirm that the out-of-scope list is stated in RWO and not duplicated in RWP.

### 1.4 Relationship to other documents

- [RWC]: concept.
- [RWP]: Record structure, identity, versions, integrity, federation, payload deletion.
- [DID-CORE], [DID-RESOLUTION]: DID documents, service entries, resolution.
- [OAIS] (ISO 14721): vocabulary for roles and the notions of submission and access.
- [RFC9110]: HTTP semantics for the HTTP binding.
- [RFC9457]: machine-readable error details.

### 1.5 Terminology

The terms *producer*, *custodian*, *consumer*, *resolver*, *Record* and *version* are used as defined in RWP. The key words MUST, MUST NOT, SHOULD and MAY are to be interpreted as described in RFC 2119.

- **Master custodian**: the custodian whose endpoint is registered in the DID document of the namespace.
- **Operation**: an abstract function offered by a custodian.
- **Option**: a parameter that narrows or modifies an operation.
- **Profile**: a named set of operations a custodian claims to implement.

---

## 2. Conformance

### 2.1 Roles

- A **custodian** claiming an RWO profile MUST implement all mandatory operations of that profile and MUST describe them via Describe.
- A **producer** or **consumer** claiming an RWO profile MUST use the operations as defined and MUST NOT rely on behaviour that is not declared by Describe.

### 2.2 Profiles

| Profile | Mandatory operations | Optional operations |
|---|---|---|
| RWO Custodian Core | Retrieve, Submit, Erase, Describe | none |
| RWO Custodian Extended | Core | any operation or option announced via Describe |

> Issue: Decide whether Submit and Erase can be claimed separately (read-only custodian, e.g. an archive after closure).

### 2.3 Master rule

There is exactly one master custodian per namespace. A custodian that is not the master MUST NOT accept Submit or Erase for that namespace and MUST answer with the error `not-master`. A non-master custodian MAY serve Retrieve for copies it holds if it declares this via Describe.

> Issue: Define whether non-master Retrieve is allowed at all, and how a consumer recognises a copy.

---

## 3. Operations (abstract)

All operations act on Records identified by RWP DIDs. Every operation has a **default behaviour** that applies when no option is given. The default behaviour MUST be identical for all custodians.

### 3.1 Retrieve

**Purpose:** return a Record, or a defined part of it.

**Default (no options):** the complete Record: all versions, the version graph, integrity evidence and metadata, in a form that allows a receiving custodian to validate and accept it via Submit without loss.

**Narrowing:** see chapter 4.

| Aspect | Definition |
|---|---|
| Input | Record DID, optional narrowing, optional options |
| Output | Representation of the selected Record or part |
| Preconditions | Caller is permitted (out of scope); Record exists |
| Postconditions | No state change in the custodian |
| Errors | `not-found`, `erased`, `not-supported`, `unauthorized` |

Requirements:

- Retrieve MUST NOT change any Record or version.
- A Retrieve without options MUST return everything needed to reproduce the Record at another custodian.
- If part of a Record has been erased, the response MUST make this visible (see Erase).

### 3.2 Submit

**Purpose:** add a version to a Record, or add a new Record, to the custodian.

Submit covers both creation and update. An update is always a **new version**. A version is never replaced.

**Default (no options):** the submitted version is validated and inserted into the version graph of the Record.

Requirements:

- The custodian MUST validate the submission against RWP (identity, schema association, payload validation, integrity evidence, version graph).
- The custodian MUST apply the rules of the Record's **schema** for allowed states and transitions. A draft or finalised version MAY follow a finalised version if the schema permits it.
- The custodian MUST NOT modify submitted content.
- The custodian MUST NOT replace or remove an existing version.
- Submit MUST accept versions that were created by another implementation and are already finalised, provided they validate. This enables transfer by Submit from the former custodian.
- Only the master custodian accepts Submit (see 2.3).

| Aspect | Definition |
|---|---|
| Input | Record representation or version representation |
| Output | Confirmation including the identifiers of the inserted versions |
| Preconditions | Custodian is master; submission valid |
| Postconditions | Version graph extended; no existing version changed |
| Errors | `invalid-submission`, `transition-not-allowed`, `not-master`, `not-supported`, `unauthorized` |

> Issue: Define idempotency. Submitting a version that already exists with identical content SHOULD succeed without effect.

### 3.3 Erase

**Purpose:** remove content as required by applicable law or policy.

RWO defines a single operation. What exactly is erased is determined by the application based on applicable law, not by RWO. The possible forms are:

- the payload only, with an erasure notice that remains;
- the complete Record.

Requirements:

- The response MUST state which form of erasure took place.
- After erasure, Retrieve MUST NOT return erased content and MUST signal the erasure (`erased`) rather than `not-found`.
- Erase of payloads MUST be consistent with the payload deletion requirements of RWP.
- Only the master custodian accepts Erase (see 2.3).

| Aspect | Definition |
|---|---|
| Input | Record DID, optional narrowing |
| Output | Description of the erasure performed |
| Postconditions | Content removed; erasure notice present as far as the form requires |
| Errors | `not-found`, `not-master`, `not-supported`, `unauthorized` |

> Issue: Confirm that "erased" status is required even where the complete Record is erased, and clarify the interaction with the retention of the erasure notice.

### 3.4 Describe

**Purpose:** let a client learn what a custodian offers.

See chapter 5.

### 3.5 Transfer

RWO defines no Transfer operation. A transfer of Records between custodians is composed of existing operations:

- the future custodian performs Retrieve at the former custodian, **or**
- the former custodian performs Submit at the future custodian.

Neither suffices for a change of responsibility (see 1.3).

---

## 4. Addressing and Options

### 4.1 Namespace resolution

The endpoint of a custodian is resolved **per namespace** via the DID document, as defined in RWP. A client MUST determine the master custodian by resolving the DID and reading the service entry of type defined in chapter 5.

> Issue (RWP): State explicitly in RWP that endpoints are resolved at namespace level and add a "how to resolve" procedure.

Consequence: all Records of a namespace have one master custodian at any time. An endpoint change applies to the whole namespace. A partial move of a namespace is not possible. The organisational requirement that all Records are available at the new custodian before the switch is out of scope (see 1.3).

### 4.2 Narrowing levels

An operation MAY be narrowed using the structure of the DID.

| Level | Selects | Applicable to |
|---|---|---|
| Record | the Record with all versions | Retrieve, Submit, Erase |
| Version | one version | Retrieve, Erase |
| Component | metadata, payload or integrity evidence of a version | Retrieve, Erase |
| Content | a part of the payload | Retrieve |

Requirements:

- With no narrowing, the level is Record.
- A custodian MUST announce supported narrowing levels via Describe.
- A request for an unsupported narrowing MUST be answered with `not-supported`.

> Issue: Map narrowing to DID path, query and fragment components consistently with RWP chapter 4.

### 4.3 Options

Options modify an operation (for example the representation format). Options are announced via Describe. A custodian MUST ignore no option silently: an unknown or unsupported option MUST lead to `not-supported`.

---

## 5. Capability Description

### 5.1 Purpose

Describe makes a custodian self-describing. A client reads the description and learns which profile, operations, narrowing levels and options are supported.

### 5.2 Service entry in the DID document

The DID document of a namespace SHOULD contain a service entry that points to the custodian. Proposed `type`: `RwoCustodian`.

```json
{
  "service": [{
    "id": "did:rwp:example.org#custodian",
    "type": "RwoCustodian",
    "serviceEndpoint": "https://custodian.example.org/rwo/"
  }]
}
```

> Issue: Align with the planned DID document extension (optional service endpoints, two mandatory, one of them the capability output).

### 5.3 Capability description document

The description is a JSON-LD document. Mandatory content:

1. The RWO profile and version the custodian conforms to.
2. The operations offered.
3. The location and binding of each operation.

Optional content: supported narrowing levels, supported options, optional operations, role (master or copy).

```json
{
  "@context": "https://recordweb.github.io/rwo/context.jsonld",
  "type": "RwoCapabilities",
  "conformsTo": "https://recordweb.github.io/rwo/#custodian-core",
  "version": "0.0.1",
  "operations": {
    "retrieve": {
      "href": "https://custodian.example.org/rwo/records",
      "binding": "http",
      "narrowing": ["record", "version", "component", "content"]
    },
    "submit":   { "href": "https://custodian.example.org/rwo/records", "binding": "http" },
    "erase":    { "href": "https://custodian.example.org/rwo/records", "binding": "http" },
    "describe": { "href": "https://custodian.example.org/rwo/", "binding": "http" }
  }
}
```

> Issue: Vocabulary. Options: own JSON-LD context, informed by DCAT (`conformsTo`, `endpointDescription`). PROV is not used for capabilities.

### 5.4 Requirements

- Describe MUST be available without prior knowledge beyond the DID.
- The description MUST be consistent with actual behaviour.
- A client MUST NOT assume an optional operation unless it is announced.

---

## 6. Errors

### 6.1 Error types

Errors are identified by a type URI in the RWO namespace and returned as machine-readable problem details [RFC9457].

| Type | Meaning |
|---|---|
| `not-found` | Record, version or component does not exist |
| `not-master` | This custodian is not the master for the namespace |
| `invalid-submission` | Submission violates RWP, schema or integrity rules |
| `transition-not-allowed` | State transition not permitted by the schema |
| `not-supported` | Operation, narrowing or option is not offered (see Describe) |
| `erased` | Content or Record has been erased (erasure notice) |
| `unauthorized` | Access not permitted (policy out of scope) |

### 6.2 Requirements

- A custodian MUST distinguish `erased` from `not-found`.
- Error responses MUST contain the type; `title` and `detail` SHOULD be present.

> Issue: Decide the base URI for error types, for example `https://recordweb.github.io/rwo/errors#`.

---

## 7. HTTP Binding

### 7.1 General

The HTTP binding maps the abstract operations to HTTP semantics [RFC9110]. Error responses use `application/problem+json` [RFC9457].

### 7.2 Mapping (proposal)

| Operation | HTTP method | Notes |
|---|---|---|
| Retrieve | `GET` | Safe and idempotent; narrowing via path/query |
| Submit | `POST` | Adds a version; a version is never replaced, therefore no `PUT` |
| Erase | `DELETE` | Response describes the form of erasure |
| Describe | `GET` | Returns the capability description |

### 7.3 Error mapping (proposal)

| Error type | HTTP status |
|---|---|
| `not-found` | 404 |
| `not-master` | 421 or 409 |
| `invalid-submission` | 422 |
| `transition-not-allowed` | 409 |
| `not-supported` | 501 or 400 |
| `erased` | 410 |
| `unauthorized` | 401 or 403 |

> Issue: Choose between 421 and 409 for `not-master`, and between 501 and 400 for `not-supported`. Verify the semantics against RFC 9110 before fixing the mapping.

### 7.4 Content types and headers

> Issue: Define media types for Record representations, version negotiation and cache behaviour. Finalised versions are immutable and SHOULD be cacheable.

---

## 8. Security and Privacy Considerations

- Authentication and authorisation are out of scope and left to the deployment.
- Erase interacts with legal obligations. The form of erasure MUST be visible to consumers.
- A client MUST validate Records it retrieves according to RWP, regardless of which custodian delivered them.
- Describe documents reveal capabilities. Deployments MAY restrict them but MUST NOT publish inaccurate descriptions.

---

## Annex A. Examples

> To be added: Retrieve without options, Retrieve with narrowing to a payload part, Submit of a new version, Submit of a transferred finalised version, Erase with payload-only form, error responses.

## Annex B. Open Issues Summary

1. State endpoint resolution per namespace and a "how to resolve" procedure in RWP.
2. Confirm erase response semantics and the `erased` signal.
3. Profile split for read-only custodians.
4. Treatment of non-master copies.
5. Idempotency of Submit.
6. Mapping of narrowing to DID components.
7. DID document service entry and capability vocabulary.
8. HTTP status choices for `not-master` and `not-supported`.
9. Media types, versioning and caching.
10. Final name and abbreviation of this specification (working name: RWO).

## References

- [RWC] RecordWeb Concept. https://recordweb.github.io/rwc/
- [RWP] RecordWeb Protocol. https://recordweb.github.io/rwp/
- [DID-CORE] Decentralized Identifiers (DIDs) v1.0. https://www.w3.org/TR/did-1.0/
- [DID-RESOLUTION] DID Resolution v1. https://www.w3.org/TR/did-resolution-1.0/
- [RFC9110] HTTP Semantics. https://www.rfc-editor.org/rfc/rfc9110.html
- [RFC9457] Problem Details for HTTP APIs. https://www.rfc-editor.org/info/rfc9457/
- [OAIS] ISO 14721, Reference Model for an Open Archival Information System.
