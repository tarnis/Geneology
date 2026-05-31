# Ancestor Dossiers — schema & research protocol

Generated from `export-BloodTree.ged` (Geni.com export). One JSON file per
**direct ancestor** of the proband (Matthew Evan Hyner). Collateral branches
(siblings, cousins, in-laws, spouses' families) are intentionally excluded.

## Files
- `I<id>.json` — one dossier per ancestor, named by GEDCOM/Geni ID.
- `_index.json` — manifest of everyone: id, name, sex, generation, birth, filename.
- `_README.md` — this file.

## Dossier schema
```json
{
  "id": "I6000000...",            // GEDCOM/Geni ID — the stable merge key
  "gedcom_xref": "@I6000000...@",
  "schema_version": 1,
  "names": ["Benjamin Hiner", "Benjamin Hyner"],
  "sex": "M",
  "generation": 3,                // 0 = proband, 1 = parents, 2 = grandparents...
  "relationships": {              // ID references only; resolve via _index.json
    "parents":  [{"id": "...", "name": "Michael Hiner"}],
    "spouses":  [{"id": "...", "name": "Beckie Marcus"}],
    "children": [{"id": "...", "name": "Irving Hyner"}]
  },
  "facts": [ /* append-only ledger, see below */ ],
  "research_notes": [ {"text": "...", "source": "...", "date_added": "..."} ],
  "last_updated": "2026-05-20"
}
```

## The fact ledger (append-only)
Every fact is one entry. **Never overwrite or delete an existing fact** — only
append new ones, or change an existing fact's `status`. This preserves the full
research trail and lets conflicting claims coexist until resolved.

```json
{
  "field": "birth",              // birth, death, burial, residence, immigration,
                                 //   naturalization, census, occupation,
                                 //   marriage, divorce, religion, ...
  "value": "...",                // descriptive content (e.g. occupation text)
  "date": "15 MAR 1884",         // event date; keep raw GEDCOM form (ABT/BEF/AFT ok)
  "place": "Slonim, Belarus",
  "status": "established",        // see vocabulary below
  "source": "GEDCOM export (Geni.com)",
  "date_added": "2026-05-20",
  "note": "optional context"
}
```

### Status vocabulary
- **established** — from the GEDCOM, or since confirmed by a solid source.
- **possible** — a candidate found but not yet verified.
- **rejected** — investigated and disproven. Keep the entry; add
  `rejected_date` and `rejected_reason`.
- **superseded** — was held true, then corrected. Keep the entry; the
  replacement is a new fact. (rejected = *wrong/never true*; superseded =
  *understanding improved*.)

## Research protocol (for the browsing agent)
1. **Read the whole dossier first**, including `rejected` facts.
2. **One ancestor, one site per session.** Use the established facts as
   anchors (name variants, town, dates, spouse) to identify matches.
3. **Match rule:** only mark a finding `established` if town AND at least one
   family-member name line up. Otherwise add it as `possible`.
4. **Do-not-resurrect rule:** before adding any finding, check it against the
   existing facts *including rejected ones*. If it matches a `rejected` entry,
   do **not** re-add it as `possible` — note that it matched a prior rejection
   and move on, UNLESS it carries genuinely new corroborating evidence that
   could overturn the rejection (in which case add a new fact and note that it
   reopens the prior rejection).
5. **Append, never overwrite.** Every new entry gets its own `source` and
   `date_added`.
6. **Output findings as ready-to-append ledger entries** (JSON objects matching
   the fact schema) so they merge cleanly.

## Reassembly into a master tree
Each file is keyed by `id`, relationships are ID references, and the schema is
identical across files — so a later merge is a single pass: read every
`I*.json`, key on `id` (dedupes automatically), and resolve relationship refs.
