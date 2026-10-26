# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

This is meant to become the **CDIF Access profile release repo**: the published artifacts for a
`cdifAccess` building block. It is one of the `profile-*` release repos under `C:\GithubC\CDIF\`;
the canonical source is `metadataBuildingBlocks` (mBB).

> **Status (2026-10-26): draft.** Published as
> `Cross-Domain-Interoperability-Framework/profile-access-draft`. The repo holds only docs and the
> design spreadsheet (`CDIF_Access_Profile_ODRL_Classes_Properties_Description.xlsx`). The draft mBB
> blocks `_sources/odrlProperties/odrlOffer/` and `_sources/profiles/cdifProfile/cdifAccess/` are on
> mBB branch `cdif-access-draft`, not merged, with open questions marked `OPEN:`. mBB has no
> `profile-access` entry in `tools/sync_release_repos.py`'s `REPOS` list yet. The generated file
> names below are the expected ones, not existing files.

## Bootstrapping order

1. Finish the `cdifAccess` block in mBB (`_sources/profiles/cdifProfile/cdifAccess/`): it has a
   schema and an example; SHACL rules and `conformance.json` are still missing.
2. Add `("profile-access", "_sources/profiles/cdifProfile/cdifAccess", "cdifAccessStructuredSchema.json", "accessRules.shacl")`
   to `REPOS` in mBB `tools/sync_release_repos.py`, then sync (below).
3. Add the hand-authored files here: `cdifAccess-frame.jsonld`, `CDIFAccessImplementationGuide.md`,
   `README.md`, `LICENSE` (CC-BY-4.0, like the siblings), `.gitignore`.
4. Bring in `FrameAndValidate.py` from the validation repo and copy
   `.github/workflows/check-frameandvalidate.yml` from a sibling (e.g. `profile-codelist`).

## What is generated, and where to fix it

Most of the root is NOT authored here. Edit the source, never the copy:

| file | source | how it gets here |
|---|---|---|
| `cdifAccessStructuredSchema.json` | mBB `_sources/profiles/cdifProfile/cdifAccess/resolvedSchema.json` | `python tools/sync_release_repos.py --apply --repo profile-access`, run **in mBB** |
| `accessRules.shacl` | mBB `validate_shacl --emit-shapes` over the same block | same sync |
| `examples/example*.json` | mBB block's `example*.json` | same sync |
| `FrameAndValidate.py` | `Cross-Domain-Interoperability-Framework/validation` `tools/FrameAndValidate.py` | `python tools/sync_frameandvalidate.py --apply` in that repo. Carries a DO-NOT-EDIT banner; CI (`check-frameandvalidate.yml`) fails on any body drift from the normative copy |

The mBB sync does NOT touch the implementation guide, the frame, `FrameAndValidate.py`, README,
LICENSE or any release-only examples; those are maintained here.

## Branches

In the sibling repos, `main` is the current release (protected, served by GitHub Pages; changes
reach it only by pull request, so the merge is the release). New work goes on `updates` and is
tagged `v1.1.n` on release. The mBB sync writes into whatever branch is checked out, so check the
branch before running it with `--apply`.

## Commands

Dependencies (unpinned; no requirements file): `pyld`, `jsonschema`, `rdflib`, `pyshacl`.

```bash
# Frame + validate one document against the profile schema (schema and frame auto-detected beside the script)
python FrameAndValidate.py examples/<example>.json -v

# Also detect conformance from content and rewrite dcterms:conformsTo
python FrameAndValidate.py <doc.json> -v --conformance -o framed.json

# SHACL
pyshacl -s accessRules.shacl -df json-ld -f human <doc.json>
```

There is no test suite. "The tests" are FrameAndValidate `-v` plus pyshacl over every file in
`examples/`. A sync is judged by both staying where they were: all examples pass the schema, and
the SHACL violation counts are unchanged.

## Traps that report PASS while wrong (from the siblings)

- **Framing picks the node matching the frame's root `@type`.** If an example's framed `@id`
  differs from its source root `@id`, the validator is reporting on the wrong node. Root selection
  (`pick_main_entity`) is fixed in the validation repo, never here.
- **An `@id` that is not a valid IRI silently drops the node and its incoming triple** during RDF
  expansion, after the JSON Schema gate has passed. When adding an example, parse it and compare a
  property's triple count with its key count in the JSON. They must be equal.

## Example conventions (shared across CDIF profiles)

- `@context` declares prefixes explicitly (`schema`, `dcterms`, `dcat`, `prov`, ...), never `@vocab`;
  schema.org namespace is `http://schema.org/` (not `https://`).
- All schema.org properties carry the `schema:` prefix; `@type` is an array (`["schema:Dataset"]`).
- `schema:subjectOf` holds a `dcat:CatalogRecord` whose `dcterms:conformsTo` lists the profile URI
  (`https://w3id.org/cdif/<profile>/1.1`; the Access URI is not yet defined).
- Validation is open-world: never strip unknown properties from an example.
- Bare reference slots are sealed `{@id}` objects (`additionalProperties: false`, `required: ['@id']`).
