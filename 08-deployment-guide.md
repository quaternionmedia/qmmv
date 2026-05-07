# Deployment Guide

How to go from this docset to a working vault. Roughly 30–60 minutes of setup, then onboarding begins immediately.

---

## Prerequisites

- Obsidian installed (any recent version).
- A vault. New or existing — these docs can drop into either.
- Templater plugin installed (community plugin).
- *Optional but recommended:* QuickAdd plugin, Dataview plugin (Dataview only matters in the analysis phase, but install now to confirm working).

---

## Step 1 — Place the docs

Choose where the documentation lives. Two options:

**Option A — Top-level `_meta/` folder (recommended).**

```
<vault>/
  _meta/
    00-README.md
    01-schema-core.md
    ...
    99-schema-changelog.md
  Media/                       (created in step 2)
  Templates/                   (created in step 3)
  _Inbox/                      (created in step 2)
```

**Option B — Vault root.** Same files but at the top level. Visually clutters the file tree but makes the README more discoverable.

The leading underscore on `_meta/` and `_Inbox/` keeps them at the top of alphabetical sorts.

---

## Step 2 — Create the folder structure

Create the folder tree from `01-schema-core.md § Folder structure`. From a terminal at the vault root:

```bash
mkdir -p Media/Works/{Compositions,Books,Films,TV,Games,Albums,Articles,Podcasts,Comics,Performances,Other}
mkdir -p Media/{Recordings,Editions,Episodes,Productions,Collections,Series}
mkdir -p Media/People/{Composers,Authors,Directors,Performers,Conductors,Other-Creators}
mkdir -p Media/Organizations/{Ensembles,Studios,Publishers,Labels,Networks}
mkdir -p Media/Places/{Venues,Studios-Recording}
mkdir -p Media/Subjects
mkdir -p Templates
mkdir -p Attachments
mkdir -p _Inbox
```

Obsidian will see these on next focus and add them to the file explorer.

---

## Step 3 — Create the sentinel notes

These are the three notes that serve as link targets for special cases (per `01-schema-core.md § Sentinel notes`). Create each with minimal frontmatter:

**`Media/People/Composers/Anonymous.md`**

```yaml
---
kind: person
type: composer
name: "Anonymous"
created: 2026-05-06
confidence: confirmed
notes: "Sentinel note. Use as composer link for anonymous works."
---
```

**`Media/People/Composers/Traditional.md`**

```yaml
---
kind: person
type: composer
name: "Traditional"
created: 2026-05-06
confidence: confirmed
notes: "Sentinel note. Use as composer link for folk/traditional pieces."
---
```

**`Media/People/Performers/Various Artists.md`**

```yaml
---
kind: person
type: performer
name: "Various Artists"
created: 2026-05-06
confidence: confirmed
notes: "Sentinel note. Use for compilations not bundled by performer."
---
```

These let `composer:` and `artists:` always be valid wikilinks rather than null/empty special cases.

---

## Step 4 — Configure Templater

In Obsidian settings → Templater:

- **Template folder location:** `Templates`
- **Trigger Templater on new file creation:** off (you'll invoke explicitly).
- **Enable user system command functions:** on (needed for date prompts).
- **Folder Templates:** none yet (set up in step 6).

---

## Step 5 — Build the Tier 1 quick-capture template

The minimum-viable template set. Create `Templates/00-quick-capture.md`:

```markdown
<%*
const title = await tp.system.prompt("Title");
const kind = await tp.system.suggester(
  ["work", "recording", "edition", "person", "collection", "series", "org", "place", "subject"],
  ["work", "recording", "edition", "person", "collection", "series", "org", "place", "subject"]
);

const typesByKind = {
  work: ["composition", "book", "film", "tv-series", "game", "album", "article", "podcast", "comic", "performance-work", "improvisation"],
  recording: ["recording"],
  edition: ["book-edition", "film-edition", "game-edition", "recording-release", "comic-edition", "score-edition"],
  person: ["composer", "author", "director", "performer", "conductor", "creator"],
  collection: ["collection"],
  series: ["series"],
  org: ["ensemble", "studio", "publisher", "label", "network"],
  place: ["venue", "recording-studio"],
  subject: ["subject"]
};

const type = await tp.system.suggester(typesByKind[kind], typesByKind[kind]);
const source = await tp.system.prompt("Source (optional)", "");

await tp.file.rename(title);
await tp.file.move(`/_Inbox/${title}`);
-%>
---
kind: <% kind %>
type: <% type %>
title: "<% title %>"
created: <% tp.date.now("YYYY-MM-DD") %>
source: "<% source %>"
confidence: stub
needs-review: false
tags: []
---

# <% title %>

> Captured <% tp.date.now("YYYY-MM-DD") %><% source ? " — source: " + source : "" %>
```

Bind to a hotkey via Obsidian settings → Hotkeys → "Templater: Open Insert Template modal" *or* via QuickAdd if you want a single keystroke.

This is enough to start onboarding. Tier 2 templates (per-kind full schemas) can be built incrementally per `04-templater-plan.md § Build order`.

---

## Step 6 — Build the next four high-priority templates

Per the build order in `04-templater-plan.md`, build these next so classical music cataloging works end-to-end:

1. `10-work-composition.md`
2. `20-recording.md`
3. `60-person-composer.md`
4. `64-person-conductor.md`
5. `70-org-ensemble.md`

Each of these uses the schema in `01-schema-core.md` and the catalog system patterns in `07-catalog-systems.md`. Frontmatter scaffolds copy directly from those documents; Templater interpolation handles the prompts.

---

## Step 7 — Smoke test

Capture three notes to confirm everything works:

1. **A composition** — use Quick Capture: kind=work, type=composition, title="Symphony No. 9 in D minor". Confirms basic capture.
2. **A composer** — use Quick Capture: kind=person, type=composer, title="Ludwig van Beethoven". Confirms person creation.
3. **Open the composition note and add the link** — `composer: "[[Ludwig van Beethoven]]"`. Confirms the autocomplete picks it up.

If all three work and the wikilink resolves, the vault is operational.

---

## Step 8 — Begin onboarding

Per `03-onboarding-workflow.md`:

- Stage 1 captures from now on go through Quick Capture into `_Inbox`.
- Schedule a weekly time for the linking pass.
- Stage 3 enrichment happens on demand.
- Stage 4 edition notes happen when you acquire a copy.

Resist the urge to:

- Build Dataview queries.
- Pre-create stub notes for famous composers/works you don't yet care about.
- Design the analysis layer.
- Refine the schema beyond fixing actual bugs.

The first 30–60 days are about volume of capture, not perfection.

---

## Step 9 — Backups

Before serious onboarding begins:

- Confirm vault is in a backed-up location (cloud sync, git repo, Time Machine, etc.).
- Optionally: initialize a git repo at the vault root, commit the docset, and commit on a regular cadence.

The schema can survive a lost note here and there. It can't survive losing the whole vault to a sync conflict.

---

## Rollback / undo

If you decide the schema isn't working:

- Notes are plain markdown with YAML frontmatter. They survive any schema change.
- Folder reorganization is a drag-and-drop in Obsidian.
- Field renames are a search-and-replace scoped to YAML.

The cost of changing your mind is low. The cost of not starting is high. Start.
