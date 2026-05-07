# Handoff Context

This document is for the next agent (or future-you) picking up the Media Vault project. It captures *why* the schema looks the way it does, what was deliberately deferred, what's known to be unresolved, and where the bodies are buried.

If you're an agent reading this cold: read `00-README.md` first for orientation, then `01-schema-core.md` for the entity model, then come back here for the rationale and open questions before making changes.

---

## Project state at handoff

**Schema version:** 1.0.0 (finalized 2026-05-06; see `99-schema-changelog.md`).
**Phase:** Cataloging schema design — complete.
**Phase next:** Vault deployment per `08-deployment-guide.md`, then onboarding per `03-onboarding-workflow.md`.
**Decisions made:** Documented below.
**Decisions deferred:** Documented below as "Open questions."
**Code/templates written:** None yet. The Tier 1 quick-capture template body is in `08-deployment-guide.md § Step 5`. Tier 2+ templates remain to be authored per the build order in `04-templater-plan.md`.

---

## Why this schema looks the way it does

### Why three layers (Work / Recording / Edition) for music

The naive model is a single "album" or "recording" entity that conflates the composition, the performance, and the release. This breaks immediately for classical:

- "All recordings of Beethoven 9" — impossible if the composition isn't its own entity.
- "All recordings made at Jesus-Christus-Kirche" — impossible if recording metadata is buried in album metadata.
- "Distinguish 1962 LP from 2003 SACD remaster" — impossible if release format isn't its own entity.

Three layers cost more to maintain but earn their cost the first time you want any of those queries. The cost is also asymmetric — Composition + Recording is required for classical; Edition is optional and lazy-create.

For pop/rock/jazz, the album-as-work model collapses Composition+Recording back into one. The schema supports both via `type: album` (single-entity) and `type: composition` + `kind: recording` (split). Same vault, different conventions per genre.

### Why entities for People, Orgs, Places, even Subjects

The temptation: "composer is just a string, why a whole note?"

Counter-argument that won this design: The moment you want "all works by Beethoven sortable by year" or "all recordings featuring the Berlin Philharmonic" or "all recordings made at Jesus-Christus-Kirche," string-valued fields require fragile prefix-matching and fail on aliases (Beethoven vs L. v. Beethoven vs Ludwig van Beethoven). Entity notes give you single canonical referents with aliases handled, plus a place to put bio and discography prose, plus graph connectivity.

Cost: stub Person notes proliferate. Mitigation: the linking-pass workflow makes stub creation cheap and explicit; `confidence: stub` flags them as incomplete.

### Why containers (Series, Collection) are first-class

`series: "Lord of the Rings"` as a string means you can't ask "what's in the LOTR series" without scanning every note. As an entity, the Series note's `parts:` array gives you the answer in one query, and child notes can declare `part-of:` for reverse-traversal. Both directions stored redundantly because the cost is one extra field and the benefit is robustness.

Collections vs Series — a real distinction. Series are *intrinsic* (declared by creator, like a numbered series). Collections are *curated* (assembled by you, a publisher, an institution). Same data shape mostly, but the semantic difference matters: a Beethoven cycle (Karajan/BPO 1961-62) is a Collection, not a Series. The Series is "Beethoven's Symphonies" — that's intrinsic to the works themselves.

### Why `kind` AND `type`

Two-level polymorphism. `kind` lets you write queries that span types within a category ("all works regardless of medium"). `type` lets you query specifically ("only films" / "only compositions"). One field can't do both well without proliferating values.

### Why `confidence` and `needs-review`

The single biggest risk to a personal-cataloging vault is the perfectionism trap: "I'll add it once I have time to do it right." The vault dies. `confidence: stub` legitimizes incomplete notes. `needs-review` flags notes the future you should revisit. Both are queryable, so "what needs love" becomes an actual list, not a vague guilt.

### Why `_Inbox` as a folder

Capture-first workflow requires that classification *cannot* block capture. The moment you have to decide "is this a Composition or a Performance Work" at the moment of capture, you'll defer capturing and lose items. `_Inbox` is the pressure release valve. Stage 2 (weekly linking) is when classification happens, with no capture pressure.

### Why the Edition entity is optional

For most users most of the time, "I read this book" doesn't need format granularity. The Edition entity is for users who *do* care: vinyl collectors, Criterion completionists, edition-conscious readers. The schema accommodates them; it doesn't demand them. A book Work note alone with `status: finished` is a complete record for someone who doesn't care which paperback printing they read.

### Why classical music gets twenty edge cases

Because it has twenty real edge cases. Versions and revisions (Bruckner 8 in three editions). Spurious works (BWV 565). Arrangements (Pictures at an Exhibition / Ravel). Improvisation (Köln Concert) outside the work-recording model. Productions of operas as a separate layer from recordings of those productions. Period vs. modern instruments. Cadenzas. Liturgical context. Pasticcios. Most of these have analogs in other media (director's cuts, translated books, mixed-author anthologies), but classical concentrates them.

The edge cases document is comprehensive *because* an incomplete edge-case treatment is worse than none — it gives a false sense of robustness. If you encounter an edge case not covered, add it to `02-classical-edge-cases.md` rather than papering over it.

---

## Decisions made (so you don't relitigate)

| Decision | Choice | Rationale |
|---|---|---|
| Three-layer music model | Composition + Recording + Edition | Justified above |
| Pop music model | Album-as-work | Different conventions; both supported |
| Folder structure | Flat within /Works (subfolders by type only) | Avoid deep nesting; type lives in frontmatter anyway |
| Filename includes creator/year | Yes | Disambiguates in autocomplete and folder views |
| Sort-title separate field | Yes | Filenames stay natural; sorting works correctly |
| Containers (Series, Collection) as entities | Yes | Enables membership queries |
| People as entities even when stub | Yes | Single canonical referent; alias handling |
| Universal rating scale | Pending — see open questions | Default in schema: yes |
| Episode notes lazy-create | Yes | Most users don't catalog every episode |
| Production notes for staged works | Optional | Most users skip; opera collectors benefit |
| `_Inbox` as capture target | Yes | Classification cannot block capture |
| Confidence + needs-review fields | Yes | Legitimizes incompleteness |
| Two-direction container linkage | Yes (`items:` + `part-of-collection:`) | Robust to either query direction |
| Templates per type, not universal | Yes | Frictionless creation; only relevant fields shown |
| No required fields beyond minimum | Yes | Capture friction kills vaults |
| Catalog data is structured, not stringly | Yes | Hoboken's group:number, BWV's Anh./variants/sub-versions, Köchel's editions all need fields, not parsed strings. See `07-catalog-systems.md`. |
| `catalog-system` is a documented enum | Yes | Undocumented systems break Dataview groupings silently; new systems must be added to `07-catalog-systems.md` |

---

## Open questions

These need owner input. Defaults documented in the schema; flagged here as places future agents should ask about rather than silently change.

### 1. Single rating scale, or per-medium scales?

**Default in schema:** Single 1–10 scale used across all media.

**Tradeoff.** A single scale is simpler and lets you query across media ("my top-rated items overall"). A per-medium scale is more honest — a 9/10 film and a 9/10 album aren't really comparable. The cost of switching later is moderate (rename the field; existing ratings stay valid within their medium).

**Recommendation:** Stick with single scale through onboarding. Reconsider when analysis layer is designed.

### 2. Pre-vs-post enrichment edition note creation

**Default in schema:** Edition notes are stage 4 (on acquisition).

**Tradeoff.** If the user only ever cares about the work, the Edition entity is dead weight. If they later want format-conscious queries, retroactive Edition creation is tedious.

**Recommendation:** Don't pre-build editions. Add them when the question "which copy?" actually arises.

### 3. How aggressive on splitting cycle parts into separate Compositions?

**Default in schema:** Lazy promotion via `parts-promoted:` array on parent.

**Tradeoff.** Splitting (Goldberg Variation 25 as its own Composition) enables fine-grained ratings and recording links but proliferates notes. Not splitting means losing per-variation engagement data.

**Recommendation:** Split only when the user has actual recordings of just the part, or wants to write about the part specifically.

### 4. Production layer for opera/ballet — yes or no?

**Default in schema:** Optional `kind: production` defined; not required.

**Tradeoff.** Without it, the same production with multiple recording years is awkward to model. With it, simple opera tracking gets a layer the user may not need.

**Recommendation:** Skip until the user has 5+ opera recordings and starts caring about staging vs. performance distinction. Then introduce it.

### 5. What about concert attendance / live event tracking?

**Default in schema:** Not modeled.

**Tradeoff.** A live concert is its own entity (different from any of: work, recording, production). Modeling it adds another `kind`. Skipping it loses the data when the user goes to concerts.

**Recommendation:** Defer. If the user wants this, introduce `kind: event, type: concert` with date/venue/program-of-works fields. The `program:` field would be an array of Composition links; performers and conductor analogous to Recording.

### 6. Annotations / quotes / highlights — separate system or in-vault?

**Default in schema:** Not addressed.

**Tradeoff.** Many users want to clip quotes and link them to works. Inline (a `quotes:` field on the work) is fragile for long quote lists. Sibling notes (`Notes on <Work>` or `Highlights from <Work>`) are flexible but proliferate.

**Recommendation:** Defer. When introduced, prefer sibling notes with explicit `kind: annotation, type: quote | highlight | note` and `of:` linkage. Keeps the work note clean.

### 7. Tag taxonomy — when to consolidate?

**Default in schema:** Free-form tags, lowercase, hyphen-separated.

**Tradeoff.** Free-form lets the user capture without taxonomy decisions. Eventually, "magical-realism" and "magic-realism" both exist and queries miss results.

**Recommendation:** First pass at consolidation after ~3 months of capture. Use Obsidian's tag pane to identify near-duplicates, batch-rename. Don't try to design a taxonomy upfront.

### 8. Per-genre defaults for `period`, `genres`, `forms`

**Default in schema:** Open-ended, no constrained vocabulary.

**Tradeoff.** Constrained vocabulary enables clean Dataview groupings; open-ended lets the user use whatever language they think in. Mid-vault constraint changes are painful.

**Recommendation:** Stay open-ended through onboarding. When analysis layer is designed, identify the vocabulary that emerged organically and document it as a soft convention. Don't enforce.

---

## Deferred work (analysis layer)

When this phase begins (probably 100+ notes in), here's what to design:

1. **Status dashboards.** Currently-active queries per medium. "What am I in the middle of."
2. **Rating rollups.** Top-rated by year, by composer, by director, by genre. Year-in-review queries.
3. **Discovery aids.** "Recommended-by Sasha and unread." "In queue, prioritized by recommender count."
4. **Coverage queries.** "Beethoven works with no recording in vault." "Studio Ghibli films I haven't seen."
5. **MOCs.** Maps of Content for genres, periods, themes. Built *bottom-up* from existing data, not designed in advance.
6. **Tag consolidation pass.** Identify and merge near-duplicates.
7. **Dataview view library.** A `/Views/` folder with parameterized Dataview queries that compose into dashboards.
8. **Concert attendance / event modeling** (if user wants).
9. **Annotations / highlights system** (if user wants).
10. **Cross-medium queries.** "All works (any medium) created in 1979." "All works set in Moscow."

None of these should be built before the data exists. Speculative dashboards become technical debt the first time the schema evolves.

---

## Known gotchas

### Frontmatter list-of-objects syntax

Obsidian renders this:

```yaml
soloists:
  - { performer: "[[Janowitz]]", role: "soprano" }
```

…but Dataview parses it differently than:

```yaml
soloists:
  - performer: "[[Janowitz]]"
    role: "soprano"
```

Both are valid YAML. The expanded form is more readable in source; the inline form is denser. Pick one and use consistently per template. Schema docs use inline for compactness; templates can use expanded.

### Wiki-links inside YAML

```yaml
composer: "[[Beethoven]]"      # works
composer: [[Beethoven]]        # also works in Obsidian, but invalid YAML for some parsers
```

Quoted form is safer. Templates should always quote.

### Date formats

Use ISO 8601: `1962-10-15`. Year-only: `1962` (numeric). Year-month: `1962-10`. Don't use US-style `10/15/1962` — Dataview can't parse it consistently.

### Aliased filenames in `[[wikilinks]]`

`[[Symphony No. 9 in D minor - Beethoven|Beethoven 9]]` — pipe alias. Useful for prose readability. Doesn't affect frontmatter linking.

### File renames break external references

Obsidian updates internal `[[links]]` on rename. It does *not* update:
- URLs to the file (in published vaults).
- References from other vaults.
- References inside code blocks (Dataview queries).

Add the old filename to `aliases:` after every rename.

### Cyclical container risk

A Series can contain works; works can declare `part-of: series`. If a Series accidentally lists itself in `items:` or a work declares itself as `part-of:`, Dataview queries can loop. Schema doesn't prevent this; convention does. Don't do it.

### `null` vs empty string vs missing field

Dataview treats these differently:
- `field: null` — present, queryable as `where !field`
- `field: ""` — present, empty string
- (field omitted) — missing entirely, queryable as `where !field` *but* templater backfill won't trigger

Templates should include all schema fields with `null`, `""`, or `[]` defaults so backfill semantics are predictable.

---

## How to evolve the schema

When adding a new field:

1. Update `01-schema-core.md` first.
2. Update affected templates in `04-templater-plan.md`.
3. Decide: is this field retroactively required (existing notes need backfill) or new-going-forward (existing notes can have it empty)?
4. If retroactive: write a Dataview query that lists notes missing the field and add to a backfill checklist.
5. Document the change in `99-schema-changelog.md` (create when the first change happens).

When renaming a field:

1. Document the old → new mapping in changelog.
2. Use Obsidian's search/replace across vault, scoped to YAML.
3. Test with one note first.
4. Update Dataview queries in `/Views/`.

When deprecating a field:

1. Document deprecation in changelog with reason and replacement (if any).
2. Don't remove from existing notes; just stop using on new notes.
3. After ~6 months, optionally do a cleanup pass.

---

## What to ask the owner before making significant changes

1. Adding a new `kind` (not just a `type` within an existing kind).
2. Renaming a field that exists on more than one entity type.
3. Restructuring folder hierarchy.
4. Introducing required fields where none existed.
5. Changing rating scale or moving to per-medium scales.
6. Introducing analysis-layer features (dashboards, MOCs, rollups).
7. Anything that requires retroactive note migration.
8. Adding a new `catalog-system` enum value — small change but must be documented in `07-catalog-systems.md` first to avoid silent Dataview misses later.

For everything else (adding optional fields, refining edge cases, adding templates, fixing inconsistencies), proceed with documentation and move on.

---

## Style notes for documentation

The owner prefers:

- Iterative, gap-analysis-first design.
- Rationale alongside changes (not just *what* but *why*).
- Modular, composable structure.
- Explicit acknowledgment of tradeoffs and deferred decisions.
- Edge cases enumerated, not hand-waved.

Keep that tone in handoffs. Don't condense out the rationale to save space — the rationale *is* the value.
