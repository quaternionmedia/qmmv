# Schema Changelog

Versioned record of schema changes. Update on every change to `01-schema-core.md` or `07-catalog-systems.md`. Future agents and Dataview queries depend on this being accurate.

Changelog entries should record: what changed, why, and whether existing notes need backfill.

---

## v1.0.0 — 2026-05-06 — Initial schema

Cataloging-phase schema, ready for onboarding. No analysis-layer features.

**Entities defined.**

- `kind: work` with types: composition, book, film, tv-series, game, album, article, podcast, comic, performance-work, improvisation
- `kind: recording` (musical performances of compositions)
- `kind: edition` with types: book-edition, film-edition, game-edition, recording-release, comic-edition, score-edition
- `kind: episode` with types: tv-episode, podcast-episode
- `kind: production` (staged-work productions; optional layer)
- `kind: collection` (curated groupings)
- `kind: series` (intrinsic multi-part containers)
- `kind: person` with types: composer, author, director, performer, conductor, creator
- `kind: org` with types: ensemble, studio, publisher, label, network
- `kind: place` with types: venue, recording-studio
- `kind: subject` (genres, movements, themes)

**Universal base fields.** kind, type, title, sort-title, aliases, ids, created, source, confidence, needs-review, related, subjects, tags, status, acquired, rating, notes.

**Catalog system enum.** Documented in `07-catalog-systems.md`. Covers BWV (with all four editions and Anh. sections), Köchel (with all nine editions), Hoboken (with full group structure), Op., WoO, RV, HWV, WWV, Kk, Bartók's three systems (Sz/DD/BB), Scarlatti's three systems (Kk/L/P), and ~20 long-tail systems.

**Sentinel notes defined.** `[[Anonymous]]`, `[[Traditional]]`, `[[Various Artists]]`.

**Backfill required:** None (initial release).

---

## Format for future entries

```
## vX.Y.Z — YYYY-MM-DD — Short description

**Change.** What was added/renamed/removed.

**Reason.** Why.

**Affected docs.** Which files were updated.

**Backfill required.** Yes/No. If yes: which Dataview query identifies notes needing update; what the update is.

**Templates updated.** Which Templater templates were modified.

**Dataview queries affected.** Any queries in /Views/ that need updating.
```

---

## Versioning rules

- **Patch (X.Y.Z+1):** Documentation clarification; no field changes; no behavior change.
- **Minor (X.Y+1.0):** Fields added (optional, non-breaking); new types or kinds added; new edge cases documented.
- **Major (X+1.0.0):** Fields renamed or removed; required-field changes; incompatible enum changes; folder restructuring.

Any major-version change requires a backfill plan and owner approval before merging (per `06-handoff-context.md` § "What to ask the owner").
