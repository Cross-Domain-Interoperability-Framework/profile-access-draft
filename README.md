# CDIF Access (profile module, draft)

This repository holds the published artifacts for the **CDIF Access profile**: machine-actionable data access and usage conditions, expressed as [W3C ODRL](https://www.w3.org/TR/odrl-model/) policies attached to CDIF metadata records. The intent and scenarios are described in the CDIF book's [data access chapter](https://cross-domain-interoperability-framework.github.io/cdifbook/data-access/intro).

> **Status: draft, under review.** Open questions are listed in the implementation guide and marked `OPEN:` in the source schemas, including the `cdifa1` action vocabulary, the ODRL profile IRI and the conformance URI (`https://w3id.org/cdif/access/1.1` is proposed, not minted).

## Contents

- [CDIFAccessImplementationGuide.md](CDIFAccessImplementationGuide.md) — the profile: purpose, conformance, encoding conventions, and every class and property.
- `cdifAccessStructuredSchema.json` — JSON Schema (Draft 2020-12).
- `accessRules.shacl` — SHACL shapes.
- `examples/` — example records.
- [CDIF_Access_Profile_ODRL_Classes_Properties_Description.xlsx](CDIF_Access_Profile_ODRL_Classes_Properties_Description.xlsx) — the design source: ODRL classes and properties for an access/usage `Offer` policy, with cardinalities and constraints.

The schema, SHACL rules and examples are generated from the building blocks in [metadataBuildingBlocks](https://github.com/Cross-Domain-Interoperability-Framework/metadataBuildingBlocks) and synced here; edit them there, not here:

- `_sources/odrlProperties/` — generic W3C ODRL 2.2 blocks (`odrlPolicy`, `odrlRule`, `odrlAction`, `odrlConstraint`, `odrlParty`, `odrlAsset`, `odrlTerms`), and `odrlOffer`, the CDIF Access profile of the ODRL Offer policy.
- `_sources/profiles/cdifProfile/cdifAccess/` — the profile module that attaches policies to a record, via `odrl:hasPolicy` on a distribution or an `{@id}` reference in `schema:conditionsOfAccess`.

## Validation

```bash
pyshacl -s accessRules.shacl -df json-ld -f human examples/exampleCdifAccess.json
```

## License

[CC-BY-4.0](LICENSE)
