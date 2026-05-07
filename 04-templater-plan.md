# Templater Plan

Template inventory and scaffolding logic for low-friction note creation. Designed so the most common entities (composition, recording, book, film, person, edition) get one-keystroke capture and the long tail is still accessible.

---

## Plugin requirements

- **Templater** — required. Provides `<% %>` interpolation and template-prompt scripts.
- **QuickAdd** — recommended. Provides choosers and macro chains for the "pick a kind" capture flow.

---

## Template inventory

### Tier 1 — Quick capture (single template, dispatched)

One entry point: `Quick Capture`. Prompts for kind/type, creates stub in `_Inbox`.

```
TEMPLATE: 00-quick-capture.md
PROMPT: title
PROMPT: kind (suggester: work / recording / edition / person / collection / series / org / place / subject)
PROMPT: type (suggester filtered by kind)
PROMPT: source (free text, optional)
OUTPUT folder: /_Inbox/
FILENAME: <title> (raw, sanitized)
```

Frontmatter scaffold:

```yaml
---
kind: <%kind%>
type: <%type%>
title: "<%title%>"
created: <% tp.date.now("YYYY-MM-DD") %>
source: "<%source%>"
confidence: stub
---

# <%title%>

> Captured <% tp.date.now("YYYY-MM-DD") %><% sourceLine %>
```

This is the only template called during stage 1 capture. Total interaction: title → kind picker → type picker → enter. ~15 seconds.

### Tier 2 — Per-kind full templates

Used during stage 2 linking and stage 3 enrichment. One template per `type` so only relevant fields appear.

**Work templates:**
- `10-work-composition.md`
- `11-work-book.md`
- `12-work-film.md`
- `13-work-tv-series.md`
- `14-work-game.md`
- `15-work-album.md`
- `16-work-article.md`
- `17-work-podcast.md`
- `18-work-comic.md`
- `19-work-performance.md`
- `1A-work-improvisation.md`

**Recording / Edition / Episode / Production templates:**
- `20-recording.md`
- `30-edition-book.md`
- `31-edition-film.md`
- `32-edition-game.md`
- `33-edition-recording-release.md`
- `34-edition-comic.md`
- `35-edition-score.md`
- `40-episode-tv.md`
- `41-episode-podcast.md`
- `45-production.md`

**Container templates:**
- `50-collection.md`
- `51-series.md`

**People templates:**
- `60-person-composer.md`
- `61-person-author.md`
- `62-person-director.md`
- `63-person-performer.md`
- `64-person-conductor.md`
- `65-person-creator.md`

**Org / Place / Subject:**
- `70-org-ensemble.md`
- `71-org-studio.md`
- `72-org-publisher.md`
- `73-org-label.md`
- `74-org-network.md`
- `80-place-venue.md`
- `81-place-recording-studio.md`
- `90-subject.md`

### Tier 3 — Migration templates

Used to upgrade a `_Inbox` stub note to a full per-kind note.

```
TEMPLATE: 99-promote-stub.md
TRIGGER: from open note in _Inbox
ACTION: replace frontmatter with the per-type template's full schema, 
        preserving title / created / source / any user-entered fields
        moves note to correct folder based on kind/type
```

This is the stage 2 linking-pass workhorse.

---

## Template authoring conventions

### Default values

- All optional fields present in the template, default to empty (`""`, `[]`, `null`).
- `confidence: stub` on quick capture, `confidence: tentative` on full templates.
- `created` always populated via `tp.date.now`.
- `tags: []` always present so tag-completion plugins work immediately.

### Placeholder comments

Use HTML comments to signal where to fill in (Templater can strip these on save, or leave them as-is):

```yaml
composer: ""              # link, e.g. "[[Ludwig van Beethoven]]"
catalog: ""               # e.g. "Op. 125", "BWV 1080"
```

### Smart prompts

For high-value fields, prompt during template instantiation:

```
PROMPT: composer (suggester: existing /People/Composers notes + "create new")
PROMPT: composed-year (numeric, optional)
PROMPT: catalog (free text, optional)
```

If "create new" is chosen, kick off the composer template inline (Templater supports nested template invocation).

### Catalog-aware composition prompting

When the composition template resolves a composer, look up the composer's `catalog-systems-canonical:` and dispatch to the right prompt sequence. See `07-catalog-systems.md` for the full system enum and per-system formats.

```javascript
// pseudocode for Templater script
const composer = await tp.system.suggester(...);
const composerNote = await loadNote(composer);
const canonicalSystems = composerNote.frontmatter["catalog-systems-canonical"] ?? [];
const primarySystem = canonicalSystems[0] ?? null;

let catalog, catalogNumber, catalogGroup, catalogSection, catalogSuffix, catalogEdition;

switch (primarySystem) {
  case "bwv":
    catalogSection = await tp.system.suggester(
      ["main", "Anh. I", "Anh. II", "Anh. III"], 
      ["", "Anh. I", "Anh. II", "Anh. III"]
    );
    catalogNumber = await tp.system.prompt("BWV number");
    catalogSuffix = await tp.system.prompt("variant (a/b/.1/.2) or blank", "");
    catalogEdition = await tp.system.suggester(
      ["BWV3 (2022)", "BWV2a (1998)", "BWV2 (1990)", "BWV1 (1950)"],
      ["BWV3", "BWV2a", "BWV2", "BWV1"]
    );
    catalog = catalogSection 
      ? `BWV ${catalogSection} ${catalogNumber}${catalogSuffix}` 
      : `BWV ${catalogNumber}${catalogSuffix}`;
    break;
    
  case "k":
    catalogNumber = await tp.system.prompt("K. number (with letter suffix if any)");
    catalogEdition = await tp.system.suggester(
      ["K6 (1964) — most common", "K1 (1862)", "K9 (most recent)"],
      ["K6", "K1", "K9"]
    );
    catalog = `K. ${catalogNumber}`;
    break;
    
  case "hob":
    catalogGroup = await tp.system.prompt("Hoboken group (Roman, e.g. I, III, XVI)");
    catalogNumber = await tp.system.prompt("number within group");
    catalog = `Hob. ${catalogGroup}:${catalogNumber}`;
    break;
    
  case "sz":
    catalogNumber = await tp.system.prompt("Sz. number");
    catalog = `Sz. ${catalogNumber}`;
    // Optional: prompt for BB equivalent
    break;
    
  case "kk":   // Scarlatti — disambiguate from Mozart
    catalogNumber = await tp.system.prompt("Kk number");
    catalog = `Kk ${catalogNumber}`;
    break;
    
  case "op":
  case "rv":
  case "hwv":
  case "d":
  case "woo":
  case "wq":
  case "h":
  case "s":
  case "b":
  default:
    catalogNumber = await tp.system.prompt(`${primarySystem.toUpperCase()} number`);
    catalog = `${displayPrefix(primarySystem)} ${catalogNumber}`;
    break;
}
```

For composers with no canonical system (`catalog-system: none` — most contemporary, Verdi, Ravel), skip catalog prompting and rely on title + year + nickname.

### Filename generation

Each per-kind template encodes its filename pattern. Examples:

```javascript
// Composition
const composer = await tp.system.suggester(...);
const lastName = composer.split(" ").pop();
tp.file.rename(`${title} - ${lastName}`);

// Recording  
const conductor = await tp.system.suggester(...);
const conductorLast = conductor.split(" ").pop();
tp.file.rename(`${workShort} - ${conductorLast} ${year}`);

// Film
tp.file.rename(`${title} (${year})`);
```

When the prompted person doesn't exist, accept free text and skip the lookup.

### Folder targeting

```javascript
await tp.file.move(`/Media/Works/Compositions/${tp.file.title}`);
```

Each template hardcodes its destination folder. The `99-promote-stub` template uses a switch-on-kind to dispatch.

---

## QuickAdd flow (optional but recommended)

QuickAdd lets you bind one hotkey to "Quick Capture" without a Templater command-palette dance:

```
HOTKEY: Cmd+Shift+N
ACTION: Run macro "Quick Capture"
MACRO:
  1. Prompt for title
  2. Suggester: pick kind
  3. Suggester: pick type (filtered by kind)
  4. Run Templater template 00-quick-capture.md
  5. Open new note
```

---

## Build order

When setting up the vault, build templates in this order. Each unblocks an onboarding milestone.

1. `00-quick-capture.md` — unblocks stage 1.
2. `10-work-composition.md`, `20-recording.md`, `60-person-composer.md`, `64-person-conductor.md`, `70-org-ensemble.md` — unblocks classical-music cataloging end-to-end.
3. `11-work-book.md`, `61-person-author.md`, `72-org-publisher.md`, `30-edition-book.md` — unblocks book cataloging.
4. `12-work-film.md`, `62-person-director.md`, `71-org-studio.md`, `31-edition-film.md` — unblocks film cataloging.
5. `15-work-album.md`, `63-person-performer.md`, `73-org-label.md`, `33-edition-recording-release.md` — unblocks pop/rock/jazz.
6. `99-promote-stub.md` — unblocks efficient stage 2 workflow.
7. Everything else — fill in as you encounter the need.

Don't build all 35 templates before starting to use the vault. Build the first five, capture for a week, then build more.

---

## Anti-pattern: monolithic universal template

It's tempting to build one template with every possible field, conditionally rendered. Don't. The whole point of per-type templates is that you see only the fields relevant to *this* kind of thing, which makes filling them frictionless. A 200-line template with 80% empty fields kills enrichment.

---

## Anti-pattern: required fields

Templater can enforce required prompts. Don't. The schema's design — minimum-viable-note of four fields — depends on every other field being optional. Required fields reintroduce capture friction.

---

## Maintenance

Templates evolve. When the schema changes (new fields added, fields renamed, edge cases discovered):

1. Update the canonical schema in `01-schema-core.md`.
2. Update affected templates.
3. Existing notes don't need migration — frontmatter is forward-compatible. New fields appear empty on old notes. Backfill on demand during enrichment.

The exception is field renames, which break Dataview queries. Document renames in a `99-schema-changelog.md` if you start running queries.
