# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

An Obsidian vault schema for cataloging a personal media library. There is no build system, no package manager, no tests, and no CI. The "code" is Markdown documentation and Obsidian-specific configuration. The Templater plugin scripts (JavaScript) live inside Markdown files.

## Read order

For any work on this repo:

1. `00-README.md` — orientation and design principles
2. `06-handoff-context.md` — rationale for every major decision, open questions, known gotchas
3. `INDEX.md` — task-based navigation to the right doc for a specific question

For schema work: `01-schema-core.md` is the source of truth for all fields. `07-catalog-systems.md` is the source of truth for the `catalog-system` enum.

## Core architecture

### Entity model

Every note has `kind` (broad category) and `type` (specialization within kind). Both are on every note; queries can target either granularity.

```
kind: work         types: composition, book, film, tv-series, game, album, article, podcast, comic, performance-work, improvisation
kind: recording    types: recording
kind: edition      types: book-edition, film-edition, game-edition, recording-release, comic-edition, score-edition
kind: episode      types: tv-episode, podcast-episode
kind: production   types: staged-work (optional layer for opera/ballet)
kind: collection   types: collection
kind: series       types: series
kind: person       types: composer, author, director, performer, conductor, creator
kind: org          types: ensemble, studio, publisher, label, network
kind: place        types: venue, recording-studio
kind: subject      types: subject
```

### Three-tier music model

```
Composition → Recording → Edition (recording-release)
(the work)    (a performance)  (a specific release/format)
```

Pop/rock albums collapse Composition+Recording into a single `kind: work, type: album` note. Classical uses the split model. Both conventions coexist in the same vault.

### Capture workflow

`_Inbox/` is the capture-first staging area — classification never blocks capture. Notes start as stubs (`confidence: stub`) and are enriched through four stages: Capture → Linking → Enrichment → Edition (see `03-onboarding-workflow.md`).

### Containers

Series (intrinsic: creator-declared, numbered) and Collections (curated: assembled by you or a publisher) are first-class entities. Child notes link to parents via `part-of:` or `part-of-collection:`; parent notes mirror this in `parts:` or `items:` arrays. Both directions stored redundantly for query robustness.

### Catalog systems

The `catalog-system` field is a **documented enum** (`07-catalog-systems.md`). Undocumented values break Dataview groupings silently. Any new catalog system must be added to `07-catalog-systems.md` before use. The same applies to `catalog-edition` variants (e.g. BWV3, K6).

## Schema evolution protocol

When adding or changing fields:

1. Update `01-schema-core.md` first (source of truth).
2. Update affected template pseudocode in `04-templater-plan.md`.
3. Determine if existing notes need backfill (retroactive) or not (new-going-forward).
4. Document the change in `99-schema-changelog.md` using the format defined there.

For new `kind` values, field renames, folder restructuring, or changes to required fields: ask the owner first (see `06-handoff-context.md § What to ask the owner`).

## Sentinel notes

Three permanent notes must not be deleted or renamed:

- `Media/People/Composers/Anonymous.md` — use as composer link for anonymous works
- `Media/People/Composers/Traditional.md` — use for folk/traditional pieces
- `Media/People/Performers/Various Artists.md` — use for compilations not bundled by a specific performer

## Open design questions

See `06-handoff-context.md § Open questions` for eight documented open questions (rating scale, edition granularity, production layer, etc.). Do not silently resolve these — they require owner input.

## Analysis layer

Dataview queries, dashboards, MOCs, and rating rollups are explicitly deferred until ~100+ notes exist. Do not introduce these during the cataloging phase.
