# CDIF Access (profile module, draft)

This repository will hold the published artifacts for the **CDIF Access profile**: machine-actionable data access and usage conditions, expressed as [W3C ODRL](https://www.w3.org/TR/odrl-model/) policies attached to CDIF metadata records. The intent and scenarios are described in the CDIF book's data access chapter.

> **Status: draft, under review.** No schema, SHACL rules or examples are published here yet.

## Current state

- **Design source:** [CDIF_Access_Profile_ODRL_Classes_Properties_Description.xlsx](CDIF_Access_Profile_ODRL_Classes_Properties_Description.xlsx) — ODRL classes and properties for an access/usage `Offer` policy, with cardinalities and constraints.
- **Draft building blocks** in [metadataBuildingBlocks](https://github.com/Cross-Domain-Interoperability-Framework/metadataBuildingBlocks), merged to `main` in [PR #54](https://github.com/Cross-Domain-Interoperability-Framework/metadataBuildingBlocks/pull/54):
  - `_sources/odrlProperties/` — generic W3C ODRL 2.2 blocks (`odrlPolicy`, `odrlRule`, `odrlAction`, `odrlConstraint`, `odrlParty`, `odrlAsset`, `odrlTerms`), and `odrlOffer`, the CDIF Access profile of the ODRL Offer policy.
  - `_sources/profiles/cdifProfile/cdifAccess/` — the profile module that attaches policies to a record, via `odrl:hasPolicy` on a distribution or an `{@id}` reference in `schema:conditionsOfAccess`.

Open questions are marked `OPEN:` in those schemas, including the `cdifa1` action vocabulary, the ODRL profile IRI and the conformance URI (`https://w3id.org/cdif/access/1.1` is proposed, not minted).

Once the draft has been reviewed and the profile has SHACL rules, the structured schema, SHACL rules and examples will be synced here, as for the other `profile-*` repos.

## License

[CC-BY-4.0](LICENSE)
