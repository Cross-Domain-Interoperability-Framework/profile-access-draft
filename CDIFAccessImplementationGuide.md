# CDIF Access Profile — Implementation Guide

> **Draft for review (Dagstuhl, November 2026).** The schema, SHACL rules and example are draft. Open questions are collected in [Open questions](#open-questions) and marked `OPEN:` in the source schemas. The conformance URI `https://w3id.org/cdif/access/1.1`, the `cdifa1:` namespace and the ODRL profile IRI are proposed, not minted.

# Table of contents

- [Purpose and scope](#purpose-and-scope)
- [Background: access policies in CDIF](#background-access-policies-in-cdif)
- [Conformance](#conformance)
- [Provenance of the artifacts](#provenance-of-the-artifacts)
- [Encoding conventions](#encoding-conventions)
- [Dataset properties added by the CDIF Access profile](#dataset-properties-added-by-the-cdif-access-profile)
- [Class definitions](#class-definitions)
- [Annotated example](#annotated-example)
- [Open questions](#open-questions)

# Purpose and scope

The **CDIF Access profile module** (`cdifAccess`) specifies how a CDIF metadata record states the conditions for retrieving a digital object (**access**) and for everything done with it afterwards (**usage**). It expresses them as machine-actionable [W3C ODRL 2.2](https://www.w3.org/TR/odrl-model/) policies attached to the record's distributions.

Today, access policies are usually prose, written differently by each repository. A user meets a new policy structure at every repository, and an aggregator cannot filter, compare or automate across providers. This profile aims at a structured, standard expression of access policies. That gives data requestors transparent, consistent access processes that can be automated and queried across providers. It gives data custodians good-practice design patterns, and a way to process and track requests efficiently and equitably.

**In scope:**
- the standard expression of access and usage policies;
- publishing those policies with the metadata record;
- the automated execution of machine-readable policies.

**Out of scope** (useful enablers, but not specified here):
- how policies are derived;
- how data sensitivity is classified, or how secondary-disclosure risk is assessed;
- licences, legal agreements, intellectual property and copyright;
- IT and cyber security;
- consent and ethics approval.

The profile does not say *when* an ethics approval is needed. It gives the standard machine-readable policy that tells a requestor *that* one is needed before the data can be retrieved. Likewise it does not classify sensitivity: it expresses access constraints that follow from a provider's existing classification.

The profile is a **module**. It adds constraints to a CDIF record and is combined with other CDIF profiles (normally CDIF Core and Discovery).

# Background: access policies in CDIF

This section summarises the CDIF book's [Data Access Recommendations](https://cross-domain-interoperability-framework.github.io/cdifbook/data-access/intro).

## Why ODRL

CDIF recommends the [Open Digital Rights Language (ODRL)](https://www.w3.org/TR/odrl-model/) to describe access policies:

- It is an RDF-based W3C Recommendation, widely used to express permissions, obligations and conditions on digital assets.
- It can be serialised as JSON-LD.
- Its core model is small, but constraints, refinements and logical constraints make it flexible.
- It is extended through **ODRL profiles**.

This follows CDIF's principle of using existing, well-supported standards.

The book names two risks:
- ODRL does not cover every scenario. The scenarios below test what it can do now and what needs extension.
- ODRL is hard for domain scientists and curators to adopt without tooling.

## Access classifications are not enough

Many providers apply broad access classifications, for example:
- *open / restricted* (CESSDA);
- *Open / Safeguarded / Controlled* (UK Data Service);
- *Closed / Embargoed / Restricted / Open* (OpenAIRE).

These are useful for faceting search results, and a record may still carry one in prose (`schema:conditionsOfAccess`). They cannot drive an automated access process. Machine-actionable policies are finer-grained: they involve parties, actions and rules over sequenced request → access → usage workflows. CDIF therefore recommends policies built bottom-up in a standard language, rather than a top-down classification.

## Scenarios

The book develops the requirements from four scenarios:

1. **Standalone repository.** A research group holding sensitive clinical trial data publishes a single policy making its datasets available for secondary use by academic researchers, for example analysis of the full dataset inside a trusted research environment (TRE).
2. **Metadata aggregator.** An aggregator such as the CESSDA Data Catalogue harvests access conditions from many providers. Today it can only show free text. Standard policies would let it filter by access conditions, for example "users located in Germany may download".
3. **Mediated access (clearing house).** A broker mediates access to many repositories. It parses a record's policy, gathers and verifies the requestor's attributes, sends an ODRL **Request**, receives an ODRL **Offer** and files an ODRL **Agreement** for audit.
4. **Federated analytics.** A software agent analyses data that cannot move. The policy must be machine-checkable at transaction time. Actions for analytics, such as running a query over a limited set of variables, are not in the ODRL core vocabulary and need a CDIF ODRL profile.

## ODRL in brief

| ODRL class | Role | In this profile |
|---|---|---|
| **Policy** | Groups rules; typed by a subclass (Set, Offer, Agreement, Request, …) | An **Offer**: the provider's standing conditions for a digital object, naming no particular user |
| **Rule** | A **Permission** (may do), **Prohibition** (must not do) or **Duty** (must do) | Permissions, prohibitions and obligations of the Offer |
| **Action** | The operation a rule is about | ODRL core actions, plus `cdifa1:` actions for what the core vocabulary lacks |
| **Asset** | The resource a rule applies to | A distribution (or data service) of the dataset |
| **Party** | Who takes a role (assigner, assignee) | The repository is the assigner; users are scoped by constraints or PartyCollections |
| **Constraint** | A condition: left operand, operator, right operand | E.g. `odrl:spatial odrl:eq <GBR>` |

**Access vs usage conditions.** In this profile an **access condition** must be satisfied *before* the object is retrieved (for example, assent to an End User Licence). It is a permission **duty** carrying the constraint `odrl:event odrl:lt odrl:policyUsage`. Every other rule is a **usage condition**: a permission, an obligation (for example, destroy the data at the end of the project) or a prohibition (for example, do not share TRE credentials).

## Why policies attach to distributions

An ODRL rule's target is the asset it governs. CDIF recommends attaching policies to a **distribution** (a `schema:DataDownload` or `schema:WebAPI`; in DCAT terms a `dcat:Distribution` or `dcat:DataService`), not to the dataset. Different distributions of one dataset can carry different conditions: an open aggregate file and a restricted microdata file, for example. The distribution points to its policy with `odrl:hasPolicy`, and each rule's `odrl:target` points back to the distribution.

# Conformance

A record conforms to the CDIF Access profile when it:

1. declares the profile in its catalog record (`schema:subjectOf`):

   ```json
   "schema:subjectOf": {
     "dcterms:conformsTo": [
       {"@id": "https://w3id.org/cdif/access/1.1"}
     ]
   }
   ```

2. carries at least one ODRL policy, in either or both of these places:
   - `odrl:hasPolicy` on a `schema:distribution` item, inline or as an `{"@id"}` reference (preferred);
   - an `{"@id"}` reference to a published policy in the dataset's `schema:conditionsOfAccess`;

3. satisfies the JSON Schema and the SHACL rules. In particular, every inline policy must be a valid CDIF Access [Offer](#sec-offer).

Records are normally also CDIF Core records. Core types the items of `schema:conditionsOfAccess` as a string, an `{"@id"}` reference or a CreativeWork link, so an inline Offer cannot go there; it goes on the distribution.

## Validation

Two validators ship with this repository:

- **JSON Schema**: `cdifAccessStructuredSchema.json` (Draft 2020-12), generated from the source register.
- **SHACL**: `accessRules.shacl`, a self-contained shapes graph. It merges the shapes of every composing building block with the profile-level shapes.

```bash
# SHACL
pyshacl -s accessRules.shacl -df json-ld -f human examples/exampleCdifAccess.json

# JSON Schema
python -c "import json,jsonschema; jsonschema.validate(json.load(open('examples/exampleCdifAccess.json')), json.load(open('cdifAccessStructuredSchema.json')))"
```

`FrameAndValidate.py` and a JSON-LD frame (`cdifAccess-frame.jsonld`) are not yet in this repository. Until they are, validate the source document directly, as above.

Validation is **open-world**: properties the profile does not describe are allowed.

The two validators check the same constraints. Each was tested by mutating the example, and every mutation that broke a schema rule also broke a SHACL rule. SHACL has to find rules by the property that holds them, because ODRL rules are usually untyped blank nodes. The SHACL shapes are:

| Shape | Source block | Checks | Severity |
|---|---|---|---|
| `CDIFAccessDatasetShape` | cdifAccess | Dataset has a policy (distribution `odrl:hasPolicy`, or an IRI in `schema:conditionsOfAccess`) | Violation |
| `CDIFAccessHasPolicyShape` | cdifAccess | `odrl:hasPolicy` values are IRIs | Violation |
| `CDIFAccessConformsToShape` | cdifAccess | Catalog record declares `access/1.1` | Warning |
| `CDIFAccessOfferShape` | odrlOffer | Offer has the CDIF profile, a policy-level assigner and at least one permission; every permission and prohibition has a target; every permission duty carries the access-condition marker | Violation |
| `ODRLPolicyShape` and subclass shapes | odrlPolicy | Policy is an IRI with at least one rule; `odrl:conflict` value; at most one policy-level target, assigner and assignee; per-subclass requirements (Set, Offer, Agreement, Request, Ticket, Assertion, Privacy) | Violation |
| `ODRLRuleShape` | odrlRule | Every rule has exactly one action, and at most one target, output, assigner and assignee | Violation |
| `ODRLActionShape` | odrlAction | Action is an IRI, or a refined action (`rdf:value` + `odrl:refinement`) | Violation |
| `ODRLConstraintShape`, `ODRLLogicalConstraintShape`, `ODRLConstraintSlotShape` | odrlConstraint | Operands and operator present and IRIs; exactly one logical operator | Violation |
| `ODRLPartyRoleShape`, `ODRLPartyShape`, `ODRLPartyCollectionShape` | odrlParty | Parties are IRIs or PartyCollections; a refined collection names its source | Violation |
| `ODRLAssetRoleShape`, `ODRLAssetShape`, `ODRLAssetCollectionShape` | odrlAsset | Targets are IRIs or AssetCollections; a refined collection names its source | Violation |

As in the other CDIF profiles, the conformance declaration is only a Warning in SHACL but a hard failure in JSON Schema. SHACL cannot tell whether a record really meets the profile, so it does not fail one for not saying so.

# Provenance of the artifacts

The schema, SHACL rules and example are generated from the canonical source register, [metadataBuildingBlocks](https://github.com/Cross-Domain-Interoperability-Framework/metadataBuildingBlocks) (mBB). Do not edit them here; edit the source and re-sync:

| File | Source in mBB | Generated by |
|---|---|---|
| `cdifAccessStructuredSchema.json` | `_sources/profiles/cdifProfile/cdifAccess/resolvedSchema.json` | `tools/resolve_schema.py` |
| `accessRules.shacl` | `rules.shacl` of cdifAccess and every block it references, plus cdifAccess `conformance.shacl` | `tools/validate_shacl.py … --emit-shapes` |
| `examples/exampleCdifAccess.json` | `_sources/profiles/cdifProfile/cdifAccess/exampleCdifAccess.json` | copied |

All three are copied here by running `python tools/sync_release_repos.py --apply --repo profile-access` in mBB.

The building blocks are:
- `_sources/profiles/cdifProfile/cdifAccess/`: this profile module;
- `_sources/odrlProperties/odrlOffer/`: the CDIF Access profile of an ODRL Offer;
- `_sources/odrlProperties/odrlPolicy`, `odrlRule`, `odrlAction`, `odrlConstraint`, `odrlParty`, `odrlAsset` and `odrlTerms`: generic ODRL 2.2 blocks, reusable outside CDIF Access.

The design source is the spreadsheet in this repository, [CDIF_Access_Profile_ODRL_Classes_Properties_Description.xlsx](CDIF_Access_Profile_ODRL_Classes_Properties_Description.xlsx).

# Encoding conventions

These follow the shared CDIF conventions, applied to ODRL:

- **Prefixed keys and explicit prefixes, not the ODRL JSON-LD context.** Every ODRL key carries the `odrl:` prefix. The document's `@context` declares the prefixes:

  ```json
  "@context": {
    "schema": "http://schema.org/",
    "dcterms": "http://purl.org/dc/terms/",
    "odrl": "http://www.w3.org/ns/odrl/2/",
    "cdifa1": "https://w3id.org/cdif/access/profile1/",
    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
    "xsd": "http://www.w3.org/2001/XMLSchema#"
  }
  ```

  The ODRL context (`http://www.w3.org/ns/odrl.jsonld`) used in the CDIF book's examples is not used. It defines `uid`, `target`, `action` and the other bare keys, and coerces string values to IRIs.
- **IRI values are `{"@id"}` objects.** Without the ODRL context nothing coerces a string to an IRI, so every target, action, party and IRI operand is written `{"@id": "…"}`. A bare string becomes a literal and fails validation.
- **`@id` replaces ODRL `uid`.** Policies, Parties and Assets are identified by `@id`.
- **`@type` is an array**: `["odrl:Offer"]`.
- **One action per rule.** ODRL allows exactly one action per rule. Several required steps are several rules or duties.
- **Policy-level properties apply to every rule.** A target, assigner or assignee given on the policy applies to all its rules ([ODRL composition](https://www.w3.org/TR/odrl-model/#composition)). CDIF asks for the assigner at policy level and the target on each rule.

# Dataset properties added by the CDIF Access profile

## schema:Dataset {#sec-schema-dataset}

The profile adds no new class to the dataset. It constrains the catalog record and the distributions, and adds `odrl:hasPolicy` to distributions.

### @context

- **Cardinality:** Optional (the record's context)
- **Content:** object
- **Description:** Declares the `odrl`, `cdifa1`, `rdf` and `xsd` prefixes, alongside the prefixes the record already uses. If declared, each must be bound to the IRI given in [Encoding conventions](#encoding-conventions).

### schema:subjectOf

- **Cardinality:** Required
- **Content:** catalog record object
- **Description:** The metadata record about the dataset. Its `dcterms:conformsTo` array must include `{"@id": "https://w3id.org/cdif/access/1.1"}`. **OPEN:** the URI is not minted. 1.1 follows the other CDIF modules; a new profile might start at 1.0.

### schema:conditionsOfAccess

- **Cardinality:** Optional; required if no distribution has `odrl:hasPolicy`
- **Content:** array of string | `{"@id"}` reference | CreativeWork link (item shape from CDIF Core)
- **Description:** Prose access conditions and/or references to the ODRL policies that formalise them. In a CDIF Access record an `{"@id"}` item should be the `@id` of an ODRL Offer, inline on a distribution or published elsewhere. Neither JSON Schema nor SHACL can tell a policy reference from any other IRI here. Keeping a prose summary alongside the reference helps human readers and aggregators that do not process ODRL.

### schema:distribution

- **Cardinality:** Optional; at least one item needs `odrl:hasPolicy` if `schema:conditionsOfAccess` has no policy reference
- **Content:** array of `schema:DataDownload` | `schema:WebAPI` (from CDIF Core)
- **Description:** The digital objects or services through which the dataset is obtained. This is the preferred place for policies, because each distribution can carry its own conditions.

#### odrl:hasPolicy (on a distribution item)

- **Cardinality:** 1..* when present
- **Content:** array of `{"@id"}` reference | [Offer](#sec-offer)
- **Description:** The ODRL policies governing access to and use of this distribution. An inline Offer must have an `@id`; a reference points to an Offer published elsewhere. The rules' `odrl:target` should be this distribution's `@id`. **OPEN:** following the book, `odrl:hasPolicy` is not declared on the Dataset itself; confirm.

# Class definitions

## Offer {#sec-offer}

The CDIF Access profile of an [ODRL Offer](https://www.w3.org/TR/odrl-model/#policy-offer): the baseline access and usage conditions a provider sets for a digital object, without naming who they apply to. Naming a specific user makes it an ODRL Agreement, which the mediated-access scenario files after a request is granted. Built on the generic ODRL Policy block (`odrlPolicy`); CDIF adds the profile, the policy-level assigner, the rule targets and the access-condition marker.

### @id

- **Cardinality:** Required
- **Content:** string (IRI)
- **Description:** Globally unique, resolvable IRI for the Offer (ODRL `uid`). A resolver should be able to return `application/ld+json`, not only an HTML page.

### @type

- **Cardinality:** Required
- **Content:** array of string; must include `"odrl:Offer"`
- **Description:** The policy subclass.

### odrl:profile

- **Cardinality:** Required
- **Content:** `{"@id": "https://w3id.org/cdif/access/profile1/cdifa1.ttl"}` (or the compact form `cdifa1:cdifa1.ttl`)
- **Description:** The CDIF Access ODRL profile, which supplies the `cdifa1:` actions the ODRL core vocabulary lacks. **OPEN:** nothing is published at this IRI yet, and it names a `.ttl` file where ODRL expects a profile identifier.

### odrl:assigner

- **Cardinality:** Required (policy level)
- **Content:** `{"@id"}` reference | [Party](#sec-party) | [PartyCollection](#sec-partycollection)
- **Description:** The party issuing the Offer, normally the repository or data custodian. ODRL also accepts an assigner on every rule instead; CDIF asks for one at policy level. The spreadsheet does not yet list an assigner.

### odrl:target

- **Cardinality:** Optional (policy level)
- **Content:** `{"@id"}` reference | [Asset](#sec-asset) | [AssetCollection](#sec-assetcollection)
- **Description:** An asset all rules apply to, if given once at policy level. CDIF still requires a target on every permission and prohibition.

### odrl:assignee

- **Cardinality:** Optional (policy level)
- **Content:** `{"@id"}` reference | Party | PartyCollection
- **Description:** A party all rules apply to. An Offer does not normally name individual users; a PartyCollection can describe the eligible group.

### odrl:permission

- **Cardinality:** Required, 1..*
- **Content:** array of [Permission](#sec-permission)
- **Description:** Actions a user may perform on the target.

### odrl:obligation

- **Cardinality:** Optional, 0..*
- **Content:** array of [Duty](#sec-duty)
- **Description:** Actions every user of the target must perform, e.g. destroy the data at the end of a project. **OPEN:** the spreadsheet gives 1..*; drafted as 0..* because an Offer with only permissions is common.

### odrl:prohibition

- **Cardinality:** Optional, 0..*
- **Content:** array of [Prohibition](#sec-prohibition)
- **Description:** Actions a user must not perform, e.g. share TRE credentials.

### odrl:inheritFrom

- **Cardinality:** Optional
- **Content:** `{"@id"}` reference, or array of them
- **Description:** Parent policies whose rules this policy inherits. This supports the book's reusable-policy pattern: a repository-wide base policy refined per dataset.

### odrl:conflict

- **Cardinality:** Optional, at most one
- **Content:** `{"@id"}`: one of `odrl:perm`, `odrl:prohibit`, `odrl:invalid`
- **Description:** How conflicts between permissions and prohibitions are resolved.

## Permission {#sec-permission}

An [ODRL Permission](https://www.w3.org/TR/odrl-model/#permission): an action a user may perform, under its constraints, once its duties (access conditions) are fulfilled.

### odrl:target

- **Cardinality:** Required
- **Content:** `{"@id"}` reference | Asset | AssetCollection
- **Description:** The asset, normally the `@id` of the distribution carrying the policy, or of a data service. Several assets are grouped as an AssetCollection, not listed.

### odrl:action

- **Cardinality:** Required, exactly one
- **Content:** [Action](#sec-action)
- **Description:** The permitted action, e.g. `cdifa1:download` or `odrl:use`. **OPEN:** the `cdifa1:` action vocabulary is not defined, so values are not enumerated.

### odrl:assignee

- **Cardinality:** Optional, at most one
- **Content:** `{"@id"}` reference | Party | PartyCollection
- **Description:** Who may exercise the permission. Typically this is a refined PartyCollection, for example registered users (`odrl:source`) located in the UK (`odrl:refinement`). This is the form the book uses; the spreadsheet scopes users with `odrl:constraint` instead. **OPEN:** settle on one, or say when each is used.

### odrl:assigner

- **Cardinality:** Optional, at most one
- **Content:** `{"@id"}` reference | Party | PartyCollection
- **Description:** The party granting this permission, if different from the Offer's assigner.

### odrl:constraint

- **Cardinality:** Optional, 0..*
- **Content:** array of [Constraint](#sec-constraint) | [LogicalConstraint](#sec-logicalconstraint)
- **Description:** Conditions on exercising the permission, e.g. `odrl:spatial odrl:eq <GBR>`: only users located in the UK.

### odrl:duty

- **Cardinality:** Optional, 0..*
- **Content:** array of [Access condition](#sec-accesscondition)
- **Description:** Actions that must be completed before the permitted action, e.g. assent to an End User Licence before download.

### odrl:output

- **Cardinality:** Optional, at most one
- **Content:** `{"@id"}` reference | Asset | AssetCollection
- **Description:** An asset created by exercising the action, e.g. the result of a derivation.

## Access condition {#sec-accesscondition}

A permission duty that must be fulfilled **before** the object is retrieved. It is marked by the constraint `odrl:event odrl:lt odrl:policyUsage`: the duty is fulfilled before the policy is exercised ([ODRL Duty on a Permission](https://www.w3.org/TR/odrl-model/#duty-perm)). **OPEN:** both terms are non-normative in ODRL 2.2. The spreadsheet does not say what an assent duty assents to (for example which licence document); a duty `odrl:target` is one candidate.

### odrl:action

- **Cardinality:** Required, exactly one
- **Content:** [Action](#sec-action)
- **Description:** The action to complete first, e.g. `cdifa1:assent`. Several pre-conditions are several duties: ODRL allows one action per rule, although the spreadsheet gives 1..*.

### odrl:constraint

- **Cardinality:** Required, 1..*
- **Content:** array of [Constraint](#sec-constraint); must include the marker:

  ```json
  {
    "@type": ["odrl:Constraint"],
    "odrl:leftOperand": {"@id": "odrl:event"},
    "odrl:operator": {"@id": "odrl:lt"},
    "odrl:rightOperand": {"@id": "odrl:policyUsage"}
  }
  ```

- **Description:** Other constraints may sit alongside the marker.

## Duty {#sec-duty}

An [ODRL Duty](https://www.w3.org/TR/odrl-model/#duty) used as a policy **obligation** (a usage condition every user must meet) or as a **remedy** or **consequence**. It has the properties of a [Permission](#sec-permission), except `odrl:duty`, plus:

### odrl:action

- **Cardinality:** Required, exactly one
- **Content:** [Action](#sec-action)
- **Description:** What must be done, e.g. `cdifa1:datadestruction`.

### odrl:target

- **Cardinality:** Optional (recommended), at most one
- **Content:** `{"@id"}` reference | Asset | AssetCollection
- **Description:** The asset the duty concerns, normally the same distribution.

### odrl:consequence

- **Cardinality:** Optional, 0..*
- **Content:** array of Duty
- **Description:** Further duties owed if this duty is not fulfilled.

## Prohibition {#sec-prohibition}

An [ODRL Prohibition](https://www.w3.org/TR/odrl-model/#prohibition): an action a user must not perform. It has the properties of a Permission (`odrl:target` required, `odrl:action`, `odrl:assigner`, `odrl:assignee`, `odrl:constraint`, `odrl:output`), but no `odrl:duty`, plus:

### odrl:remedy

- **Cardinality:** Optional, 0..*
- **Content:** array of [Duty](#sec-duty)
- **Description:** Duties owed if the prohibition is infringed.

## Action {#sec-action}

An [ODRL Action](https://www.w3.org/TR/odrl-model/#action), in one of two forms:

- a reference: `{"@id": "cdifa1:download"}` or `{"@id": "odrl:use"}`. Actions are open IRIs, so ODRL core and profile actions both validate;
- a **refined action**, when the action itself is narrowed:

  ```json
  {
    "rdf:value": {"@id": "odrl:print"},
    "odrl:refinement": [
      {"odrl:leftOperand": {"@id": "odrl:resolution"}, "odrl:operator": {"@id": "odrl:lteq"},
       "odrl:rightOperand": {"@value": "1200", "@type": "xsd:integer"}}
    ]
  }
  ```

### rdf:value

- **Cardinality:** Required in a refined action, exactly one
- **Content:** `{"@id"}` action IRI
- **Description:** The action being refined.

### odrl:refinement

- **Cardinality:** Required in a refined action, 1..*
- **Content:** array of Constraint | LogicalConstraint
- **Description:** Constraints narrowing the action.

**Actions used in the draft.** The `cdifa1:` actions come from the spreadsheet and the book. They resolve to nothing yet (**OPEN**):

| Action | Meaning | Used as |
|---|---|---|
| `cdifa1:download` | Retrieve a copy of the digital object | permission |
| `cdifa1:assent` | Assent to terms, e.g. an End User Licence | access condition |
| `cdifa1:datadestruction` | Destroy the data, e.g. at the end of a project | obligation |
| `cdifa1:shareTRECredentials` | Share credentials for a trusted research environment | prohibition |

Where an [ODRL core action](https://www.w3.org/TR/odrl-vocab/#actionsCommon) fits (`odrl:use`, `odrl:reproduce`, `odrl:distribute`, `odrl:derive`, `odrl:delete`, `odrl:attribute`, …), prefer it.

## Constraint {#sec-constraint}

An [ODRL Constraint](https://www.w3.org/TR/odrl-model/#constraint): compares a left operand with a right operand using an operator. It is used in `odrl:constraint` on rules and in `odrl:refinement` on actions and collections.

### @type

- **Cardinality:** Optional
- **Content:** array including `"odrl:Constraint"`
- **Description:** **OPEN:** the spreadsheet lists `@type` with cardinality 1 but gives no value, so it is drafted as optional.

### @id

- **Cardinality:** Optional
- **Content:** string (IRI)
- **Description:** Needed only when a LogicalConstraint refers to this constraint by reference.

### odrl:leftOperand

- **Cardinality:** Required, exactly one
- **Content:** `{"@id"}` IRI
- **Description:** What is being constrained, e.g. `odrl:spatial` (location), `odrl:dateTime`, `odrl:purpose`, `odrl:recipient`, `odrl:event`. Open IRIs, so profile operands validate.

### odrl:operator

- **Cardinality:** Required, exactly one
- **Content:** `{"@id"}` IRI
- **Description:** The comparison. ODRL defines `odrl:eq`, `neq`, `lt`, `lteq`, `gt`, `gteq`, `isA`, `hasPart`, `isPartOf`, `isAllOf`, `isAnyOf` and `isNoneOf`.

### odrl:rightOperand

- **Cardinality:** Required unless `odrl:rightOperandReference` is given
- **Content:** literal | value object (`{"@value", "@type"}`) | `{"@id"}` reference
- **Description:** The value compared against. Use an IRI from a shared vocabulary where possible, e.g. the EU country authority list (`http://publications.europa.eu/resource/authority/country/GBR`), so the condition can be evaluated beyond the issuing repository.

### odrl:rightOperandReference

- **Cardinality:** Optional (alternative to `odrl:rightOperand`)
- **Content:** `{"@id"}` IRI
- **Description:** An IRI whose dereferenced value is the right operand.

### odrl:dataType, odrl:unit

- **Cardinality:** Optional, at most one each
- **Content:** `{"@id"}` IRI
- **Description:** The datatype (e.g. `xsd:decimal`) and unit of the right operand.

### odrl:status

- **Cardinality:** Optional
- **Content:** any
- **Description:** The current value of the left operand, for comparison with the right operand.

## LogicalConstraint {#sec-logicalconstraint}

An [ODRL LogicalConstraint](https://www.w3.org/TR/odrl-model/#constraint-logical): combines constraints with exactly one logical operator. The operands are an RDF list, each item an inline Constraint or an `{"@id"}` reference:

```json
{
  "@type": ["odrl:LogicalConstraint"],
  "odrl:or": {"@list": [ {…constraint…}, {…constraint…} ]}
}
```

| Property | Meaning |
|---|---|
| `odrl:and` | all of the constraints must be satisfied |
| `odrl:or` | at least one must be satisfied |
| `odrl:xone` | exactly one must be satisfied |
| `odrl:andSequence` | all must be satisfied, in order |

## Party {#sec-party}

An [ODRL Party](https://www.w3.org/TR/odrl-model/#party): a person, organization or agent taking a role in a rule. Used for `odrl:assigner` and `odrl:assignee`.

### @id

- **Cardinality:** Required
- **Content:** string (IRI)
- **Description:** The party's identifier, e.g. an ORCID, ROR or repository IRI.

### @type

- **Cardinality:** Required (inline party)
- **Content:** array including `"odrl:Party"`, `"schema:Person"` or `"schema:Organization"`
- **Description:** ODRL makes `odrl:Party` a subclass of `schema:Person` and `schema:Organization`, so a schema.org-typed node in a party role is a Party without saying so. A `schema:Person` is validated by the CDIF person block and a `schema:Organization` by the organization block. A person may also carry `odrl:Party`. An organization may not, because the organization block only accepts schema.org types.

A party given only by reference is written `{"@id": "…"}`.

## PartyCollection {#sec-partycollection}

An [ODRL PartyCollection](https://www.w3.org/TR/odrl-model/#party-collection): a group of parties. This is the book's pattern for reusable user groups: a generic collection in `odrl:source`, narrowed by `odrl:refinement`. A finite set of common refinements could be given persistent IRIs and reused across repositories.

### @type

- **Cardinality:** Required
- **Content:** array including `"odrl:PartyCollection"`

### @id

- **Cardinality:** Required unless `odrl:source` is given
- **Content:** string (IRI)
- **Description:** Identifies the collection itself.

### odrl:source

- **Cardinality:** Required with `odrl:refinement`; at most one
- **Content:** `{"@id"}` reference
- **Description:** The collection this one is drawn from, e.g. a repository's registered users.

### odrl:refinement

- **Cardinality:** Optional, 1..* when present
- **Content:** array of Constraint | LogicalConstraint
- **Description:** Constraints selecting the members of the source, e.g. `odrl:spatial odrl:eq <DEU>`.

## Asset {#sec-asset}

An [ODRL Asset](https://www.w3.org/TR/odrl-model/#asset): the resource a rule applies to. In CDIF it is normally a distribution or data service, and is usually given by reference (`{"@id": "<distribution @id>"}`). An inline Asset has:

- `@type` (required): array including `"odrl:Asset"`;
- `@id` (required): the asset's IRI;
- `odrl:partOf` (optional): array of `{"@id"}` references to AssetCollections it belongs to.

## AssetCollection {#sec-assetcollection}

An [ODRL AssetCollection](https://www.w3.org/TR/odrl-model/#asset-collection): groups assets so one rule can target several. It has the same structure as a PartyCollection: `@type` including `"odrl:AssetCollection"`, plus either an `@id` or an `odrl:source` with optional `odrl:refinement`. The book notes that persistent collections need rules saying whether and how membership can change.

## ObjectReference {#sec-objectreference}

A sealed `{"@id": "…"}` object: a reference to a node described elsewhere. It has no other properties.

# Annotated example

[`examples/exampleCdifAccess.json`](examples/exampleCdifAccess.json) describes a restricted-access survey dataset with one CSV distribution:

- **`schema:conditionsOfAccess`** gives a prose summary and the `@id` of the Offer.
- **`schema:subjectOf`** declares `https://w3id.org/cdif/access/1.1`.
- **The distribution** `ex:dataset/3936/distribution/csv` carries the Offer inline in `odrl:hasPolicy`. The repository `https://example.org/repository` is the assigner, and the Offer declares the CDIF Access profile. Its rules all target the distribution:

  | Rule | Action | Condition | Kind |
  |---|---|---|---|
  | permission | `cdifa1:download` | constraint `odrl:spatial odrl:eq <GBR>`: users located in the UK | usage |
  | ↳ duty | `cdifa1:assent` | constraint `odrl:event odrl:lt odrl:policyUsage`: before download | **access condition** |
  | obligation | `cdifa1:datadestruction` | none | usage |
  | prohibition | `cdifa1:shareTRECredentials` | none | usage |

The core of the Offer:

```json
"odrl:hasPolicy": [{
  "@type": ["odrl:Offer"],
  "@id": "https://example.org/odrloffer/13adb49c-f641-4ac7-aa90-18651cc211ce",
  "odrl:profile": {"@id": "https://w3id.org/cdif/access/profile1/cdifa1.ttl"},
  "odrl:assigner": {"@id": "https://example.org/repository"},
  "odrl:permission": [{
    "odrl:target": {"@id": "https://example.org/dataset/3936/distribution/csv"},
    "odrl:action": {"@id": "cdifa1:download"},
    "odrl:constraint": [{
      "@type": ["odrl:Constraint"],
      "odrl:leftOperand": {"@id": "odrl:spatial"},
      "odrl:operator": {"@id": "odrl:eq"},
      "odrl:rightOperand": {"@id": "http://publications.europa.eu/resource/authority/country/GBR"}
    }],
    "odrl:duty": [{
      "odrl:action": {"@id": "cdifa1:assent"},
      "odrl:constraint": [{
        "@type": ["odrl:Constraint"],
        "odrl:leftOperand": {"@id": "odrl:event"},
        "odrl:operator": {"@id": "odrl:lt"},
        "odrl:rightOperand": {"@id": "odrl:policyUsage"}
      }]
    }]
  }],
  "odrl:obligation": [{
    "odrl:target": {"@id": "https://example.org/dataset/3936/distribution/csv"},
    "odrl:action": {"@id": "cdifa1:datadestruction"}
  }],
  "odrl:prohibition": [{
    "odrl:target": {"@id": "https://example.org/dataset/3936/distribution/csv"},
    "odrl:action": {"@id": "cdifa1:shareTRECredentials"}
  }]
}]
```

# Open questions

These are for the review. Each is also marked `OPEN:` in the source schema.

1. **Identifiers.** Neither `https://w3id.org/cdif/access/1.1` (conformance) nor the `cdifa1:` namespace (`https://w3id.org/cdif/access/profile1/`) is minted. Should the profile start at 1.0?
2. **ODRL profile.** `odrl:profile` names `cdifa1.ttl`, a file, where ODRL expects a profile identifier; nothing is published there. Is the action vocabulary an ODRL profile or a standalone controlled vocabulary? Which actions does it contain (download, assent, datadestruction, shareTRECredentials, analytics actions for federated analysis, …)?
3. **Scoping users.** Use a PartyCollection assignee with a refinement (the book's form), or a rule `odrl:constraint` (the spreadsheet's form)? Or say when each is used.
4. **Obligation cardinality.** The spreadsheet gives 1..*; the draft has 0..*.
5. **Constraint `@type`.** The spreadsheet lists it with cardinality 1 but gives no value; the draft makes it optional.
6. **The access-condition marker.** `odrl:event odrl:lt odrl:policyUsage` uses two non-normative ODRL terms. Is it the right way to distinguish access conditions from usage conditions?
7. **What an assent duty assents to.** Should the duty's `odrl:target` name the licence or terms document?
8. **Dataset-level policies.** The book argues against them, so `odrl:hasPolicy` is allowed on distributions only. The dataset can reference a policy through `schema:conditionsOfAccess`. Confirm.
9. **Assigner.** The spreadsheet does not list one; the draft requires one at policy level.
