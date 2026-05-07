# Onboarding Workflow

The four-stage capture-first approach. Designed to prevent the "perfect schema, empty vault" failure mode by separating *capture* (must be friction-free) from *enrichment* (can be optional, batched, or never happen).

---

## The four stages

| Stage | Trigger | Effort | Output |
|---|---|---|---|
| 1. Capture | New item enters awareness | 15 sec | Stub note in `_Inbox` |
| 2. Linking | Weekly batch | 5–10 min | Notes in correct folders, primary links resolved |
| 3. Enrichment | Curiosity / preparation for queries | Variable | Confirmed notes |
| 4. Edition | Acquisition of a copy | 1–2 min | Edition note linked to work |

Stages 2 and 4 are batchable. Stage 3 is *optional*. Many notes will live forever at `confidence: tentative` and that's fine.

---

## Stage 1 — Capture

Goal: get it into the vault, lose nothing.

**Trigger.** Any moment of "I should remember this": a friend recommends a book, you discover a recording, you read about a new release, you pass a director's name in an article.

**Action.** Templater command → "Quick Capture" → fills only:

```yaml
---
kind: <best-guess>
type: <best-guess>
title: ""
created: <today>
source: ""                  # who recommended; where you read about it
confidence: stub
---
```

Note saved to `/_Inbox/`. Done. Total time: 15 seconds.

**Anti-goals.**
- Don't decide which folder yet.
- Don't fill in author/composer/director — that's stage 2.
- Don't research — that's stage 3.
- Don't try to remember the exact title spelling. Approximate is fine; you'll fix it.

---

## Stage 2 — Linking pass (weekly)

Goal: classify and link, build the graph.

**Cadence.** Once a week. 5–10 minutes is enough for a normal capture rate.

**Per note in `_Inbox`:**

1. Verify the title (search Wikidata, Wikipedia, MusicBrainz, IMDb, Goodreads as appropriate). Update `aliases:` if you encounter alternate titles.
2. Add the primary creator link: composer, author, director, etc. If the person doesn't have a note yet, **create a stub Person note immediately** — name + dates + nationality is enough. Don't let missing Person notes block the linking pass.
3. Add catalog/year fields that disambiguate (catalog number for compositions, year for films, ISBN for editions).
4. Move the note to its proper folder.
5. Set `confidence: tentative`.

**Stub Person notes.** A `confidence: stub` person with three fields is fine and keeps the graph connected:

```yaml
---
kind: person
type: composer
name: "Galina Ustvolskaya"
born: 1919-06-17
died: 2006-12-22
nationality: Russian
created: 2026-05-06
confidence: stub
---
```

Resist the urge to write biographies during the linking pass. That's stage 3.

**Edge case during linking.** If you can't decide between two possible kinds (is this an album or a composition+recording?), default to the simpler model (album) and add `needs-review: true`. Future-you will have more context.

---

## Stage 3 — Enrichment (as needed)

Goal: fill in the schema fields that matter for the queries you'll eventually run.

**Triggers.**
- You're about to listen to / watch / read something and want context.
- You're about to write about it.
- You're building a Dataview query and a missing field is blocking it.
- Curiosity strikes.

**Approach.** Open the note. Fill what you know without research; leave what you don't. Set `confidence: confirmed` only when the high-value fields are filled (composer + catalog + year for compositions; director + year + studio for films; etc.).

**High-value fields by kind.**

| Kind | High-value fields |
|---|---|
| Composition | composer, catalog, catalog-system, form, period |
| Recording | work, conductor/performers, ensemble, recording-type, recorded-start |
| Book | authors, original-language, first-published, form |
| Film | directors, release-year, country, runtime |
| TV | creators, networks, first-aired, seasons-count |
| Album | artists, released, label |
| Edition | of, publisher/label, release-year, format-fields, acquired |

Don't try to fill the full schema. The *full* schema exists so it doesn't constrain edge cases, not so every field gets populated.

---

## Stage 4 — Edition note (when you acquire a copy)

Goal: track what you actually own.

**Trigger.** You buy, borrow, stream, or otherwise acquire access to a manifestation of a work.

**Action.** Templater → appropriate edition template (book-edition, film-edition, recording-release, game-edition). Link `of:` to the work or recording. Fill `acquired`, `format-fields`, `location`. Set `confidence: confirmed`.

**Skip the Edition note when:**
- You're tracking a work you've engaged with but don't own a specific copy of (e.g., heard once on the radio).
- You don't care about format granularity yet. The work note alone can carry "I've read this" via `status: finished`.

**Make the Edition note when:**
- Multiple editions exist and you want to distinguish (Criterion vs. streaming; first edition vs. paperback reprint; original mastering vs. remaster).
- Provenance matters (signed copy, gift, rare find).
- Format matters for your engagement (vinyl listening room vs. car streaming).

---

## What to do when…

### …you're not sure what `kind` something is

Default to the simpler interpretation. You can always refactor later. Most "is this a work or a recording?" confusions resolve toward `work` (the abstract thing) — the recording entity is for when you specifically want to track the *performance*.

### …you discover an item is part of a series after the fact

Open the existing note. Add `part-of:` link. Create or update the Series note with the item in `parts:`. The two-way linkage is intentional — neither direction should be the only path.

### …you bought a box set with 50 items

1. Create the Collection note first, with `items:` listing what's in it (filenames or links — links resolve later).
2. Use stage 1 capture for each work/recording in the set, batched.
3. Stage 2 linking pass classifies them and confirms links to the Collection.
4. Stage 4 — you may want a single Edition note for the box set itself (if it's a physical box) and skip per-item editions. Note the box set's catalog number, label, release year on the Collection note's release fields.

### …a Person needs splitting (two people, same name)

Rename the existing note with disambiguator suffix (`Robert Schumann` → `Robert Schumann (composer)`), create the second note (`Robert Schumann (psychiatrist)`). Update `aliases:` on both. Obsidian's rename-with-update-links handles the link-fixing.

### …you find an error in a confirmed note

Just fix it. Optionally bump `needs-review: true` if you're not sure about the correction; clear it once you've verified.

### …you want to abandon a note (decided not to track this)

Move to `/_Archive/` (create if needed). Don't delete — broken links are worse than orphan notes. The note is still link-able and discoverable in graph view.

---

## Cadence summary

```
Daily        → Capture (whenever it happens, anywhere)
Weekly       → Linking pass on _Inbox
On-demand    → Enrichment when curious or query-blocked
On-acquire   → Edition note
Monthly      → (Future) Tag consolidation, MOC review
Yearly       → (Future) Schema-evolution review, deferred-features audit
```

The monthly and yearly cadences are placeholders for the analysis-layer phase. Don't run them yet; there's nothing to consolidate or review.

---

## Anti-patterns to watch for

1. **Capturing into the right folder instead of `_Inbox`.** This is the most common drift. It feels efficient but reintroduces classification friction at capture time. The vault dies that way. Capture into `_Inbox`. Always.
2. **Refusing to create stub Person notes during linking.** "I'll come back and write up Beethoven properly later" leaves a broken link for weeks. Stub it. Five fields. Move on.
3. **Filling all 40 fields on every note.** The schema's job is to accommodate the deepest needs, not require them. Fill what matters; leave the rest empty.
4. **Building Dataview queries before there's data.** Resist for the first month minimum. The queries you imagine wanting and the queries you actually want diverge significantly once the data exists.
5. **Decision paralysis on edge cases.** When stuck between two valid models (split this work into versions or not? promote this cycle part to its own note or not?), pick the less-effort option and `needs-review: true`. Refactoring later is cheap; not capturing the data isn't.
