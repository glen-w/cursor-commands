# Documentation Maintenance (/docs)

Refactor and update project documentation so it matches the current codebase and the documentation architecture.

Run from the workspace root.

Do not modify code during this step unless explicitly requested. After completion, summarize what documentation was updated, which authority boundaries were established, and any remaining documentation gaps or ambiguities.

## Project card (fill before first use)

| Key | Value |
| --- | --- |
| Contract files | `__DOC_CONTRACTS__` |
| Surfaces map (if any) | e.g. `docs/dev/docs_architecture.md` |
| Package to walk | `__SRC_PACKAGE__` |

Skip this command on projects that have only a README and no layered docs — do a light README parity pass instead of inventing a contract tree.

---

## Documentation model (must enforce)

Docs are structured into explicit layers:

- **CONTRACT** — owns invariants, schemas, support policy, and rule definitions.
- **GUIDE** — owns user/developer flows and examples; may summarize contracts briefly, but may not define rules.
- **ARCHITECTURE** — owns system shape, boundaries, and extension points; defers to contracts for invariants.
- **PRODUCT** — owns roadmap, vision, and planning/status material.

### Hard rules

- Every major concept must have one authoritative home.
- Guides must not define rules.
- Architecture docs must not define rules.
- Runtime/ops docs must not define invariants or support policy; they may only describe runtime behavior and then link to contracts.
- If a guide, architecture doc, or runtime doc contains normative language for layout, schemas, or support policy, move or delete it and replace it with a short summary plus a link to the authoritative contract.
- Do not create new contract docs lightly; prefer extending an existing authoritative contract.
- Do not document speculative or unsupported surfaces as shipped behavior.
- Shipped delivery history belongs under `docs/archive/` with an **Archived / superseded** banner.

### Lint rules (must fail `/docs`)

- **Concept uniqueness:** fail if the same core concept is normatively defined in more than one CONTRACT doc.
- **GUIDEs:** fail if any GUIDE contains “must”, “required”, or “invariant” language that defines behavior instead of summarizing a CONTRACT doc.
- **ARCHITECTURE:** fail if the architecture doc defines behavior or invariants instead of describing structure and boundaries.
- **TERMS:** fail if `docs/TERMS.md` (when present) introduces new meanings or rule text instead of acting as a non-authoritative index.
- **Archive:** fail if a live index row presents an archived plan as current product guidance; fail if a file under `docs/archive/` lacks an **Archived / superseded** banner.

---

## 1. Classify docs by role

Map each core doc to CONTRACT / GUIDE / ARCHITECTURE / PRODUCT using the model above. Do **not** add `Type:` / `Authority:` headers to the files — keep ownership implicit via structure, titles, and cross-links.

Apply or verify `__DOC_CONTRACTS__` plus user/dev guides, architecture, and product/roadmap notes. `README.md` stays an entry guide: link deeper, avoid duplicating normative rules.

On greenfield with almost no docs: create a minimal set (`README.md` + one ARCHITECTURE + one CONTRACT) rather than a large doc tree.

---

## 2. Align docs with code

- Walk `__SRC_PACKAGE__` and confirm documented modules match reality.
- Update CLI / UI entrypoints in README when they change.
- Remove stale references to removed APIs and unsupported surfaces.
- Validate examples against the current supported surfaces (import paths, flags, compose service names).

---

## 3. README as entry guide

Keep only: brief description, primary entrypoint, secondary/programmatic entrypoints, a minimal golden path, and grouped links to deeper docs. Remove duplicated contract text.

---

## 4. Finish

- List docs created/updated.
- List authority boundaries enforced.
- List remaining gaps or ambiguities.
- Do not change application code unless the user asked.
