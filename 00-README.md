# Media Vault — Schema & Onboarding Documentation

**Version:** 1.0.0 — finalized 2026-05-06
**Status:** Cataloging-phase schema, ready for deployment and onboarding. Analysis-layer features deferred.
**Audience:** Vault owner; future Claude agent picking up the project.
**Scope:** Entity model, frontmatter schemas, classical-music edge cases, onboarding workflow, Templater plan, deployment guide. No Dataview views, dashboards, or rollups yet.

---

## What this is

An Obsidian vault schema for cataloging a personal media library across:

- **Compositions, books, films, TV, games, albums, articles, podcasts, comics, performance works** (the abstract works).
- **Recordings** (specific musical performances of compositions).
- **Editions** (specific physical or digital manifestations — book printings, film releases, recording masterings, game platform releases).
- **Episodes, collections, series** (containers and sub-units).
- **People, organizations, places, subjects** (the network of context).

The schema is designed for *robust onboarding first*. Every entity can be created as a stub with three required fields and enriched later. Nothing forces premature completeness.

## What this is not (yet)

- No analysis layer: no rating rollups, year-in-review queries, currently-active dashboards, by-creator aggregations, or MOCs.
- No tag taxonomy: tags are free-form for now and will be consolidated when patterns emerge.
- No quotes/highlights/annotations system: these are a sibling concern, deferred.
- No concert-attendance or live-event modeling.

These layer cleanly on top of the cataloging schema once data is in place.

---

## Document map

```
INDEX.md                              — quick-reference table of contents (by task, by entity)
00-README.md                          — this file
01-schema-core.md                     — folder structure, base frontmatter, all entity schemas
02-classical-edge-cases.md            — 20 edge cases specific to classical repertoire
03-onboarding-workflow.md             — capture-first workflow, four-stage enrichment
04-templater-plan.md                  — template inventory, prompts, scaffolding logic
05-naming-conventions.md              — filename patterns, sort-title rules, disambiguation
06-handoff-context.md                 — for future agent: design rationale, deferred work, open questions
07-catalog-systems.md                 — authoritative reference for BWV / Köchel / Hoboken / etc.
08-deployment-guide.md                — step-by-step vault setup
99-schema-changelog.md                — versioned record of schema changes
```

**Read order for a new vault owner:** 00 → 08 (deploy) → 03 (workflow) → start capturing. Reference 01, 02, 05, 07 as needed during capture.

**Read order for a returning agent:** 00 → 06 → INDEX → topic-specific docs.

**Reference docs (consult as needed):** 01 (every field), 02 (classical edge cases), 05 (naming), 07 (catalog systems).

---

## Core design principles

These are restated in `01-schema-core.md` but worth surfacing here:

1. **Entities, not folders, are the structure.** Folders are scaffolding; the graph lives in frontmatter links.
2. **Every entity has stable identity before rich metadata.** Stub now, enrich later.
3. **Polymorphism via `type`, shared base via `kind`.** Both fields on every note; queryable at either granularity.
4. **Containers (Series, Collections) are first-class.** Not strings inside child notes.
5. **Acquisition is separate from the work.** Editions are their own entities.
6. **Capture is frictionless or the vault dies.** `_Inbox` exists so classification never blocks capture.

## The three-tier model (musical repertoire)

```
Composition  ──────►  Recording  ──────►  Edition (Recording-Release)
(the work)            (a performance)      (a specific release/format)

Beethoven 9     →    Karajan/BPO 1962  →   1963 DG LP
                                     │
                                     ├──►  2003 SACD remaster
                                     └──►  Qobuz hi-res stream
```

For non-musical media:

```
Work         ──────►  Edition
Stalker (1979)  →     Criterion 4K UHD (2024)
                ├──►  Mosfilm DVD
                └──►  Streaming on Criterion Channel
```

Episodes (TV/podcast) are children of a Work. Productions (opera/ballet stagings) sit between Work and Recording for staged repertoire.

## Naming convention summary

| Entity | Pattern | Example |
|---|---|---|
| Composition | `Title - Composer` | `Symphony No. 9 in D minor - Beethoven` |
| Recording | `Work-short - Conductor/Performer Year` | `Beethoven 9 - Karajan 1962` |
| Book | `Title - Author` | `The Master and Margarita - Bulgakov` |
| Film | `Title (Year)` | `Stalker (1979)` |
| TV series | `Title (Network, Year)` | `The Wire (HBO, 2002)` |
| Game | `Title (Year)` | `Disco Elysium (2019)` |
| Album | `Title - Artist (Year)` | `Kid A - Radiohead (2000)` |
| Edition | `<Work> - <distinguishing>` | `Stalker (1979) - Criterion BD` |
| Person | `Full Name` | `Andrei Tarkovsky` |

Full rules in `05-naming-conventions.md`.

---

## Open questions / design choices flagged for the owner

1. **Single rating scale across all media, or per-medium scales?** Default in schema: single scale. Tradeoff in `06-handoff-context.md`.
2. **Edition granularity.** Schema supports aggressive splitting; usage doesn't require it. Default to lazy (skip Edition notes until you care).
3. **Production notes for staged works.** Optional layer. Most users skip; collectors of opera recordings benefit.
4. **Splitting cycle parts into individual Compositions.** Promote on demand only.

---

## Quick start

1. Read `08-deployment-guide.md` and follow it. (~30–60 minutes.)
2. Build the Tier 1 quick-capture template from `04-templater-plan.md`.
3. Start capturing into `_Inbox` per `03-onboarding-workflow.md`.
4. Run weekly linking pass.
5. Don't build any Dataview queries yet. Resist for at least 30 days.

For a returning agent, see `INDEX.md` for task-based navigation.
