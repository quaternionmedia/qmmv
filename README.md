# qmmv — Media Vault

An [Obsidian](https://obsidian.md) vault schema for cataloging a personal media library. Covers classical music compositions, recordings, and editions; books, films, TV, games, albums, articles, podcasts, and comics; people, organizations, places, and subjects.

The schema is designed for frictionless capture first — every entity can be created as a stub with three fields and enriched later.

---

## Fork and set up

### 1. Fork and clone

Fork this repo on GitHub, then clone your fork:

```bash
git clone git@github.com:<your-username>/qmmv.git
```

### 2. Open as an Obsidian vault

In Obsidian: **Open folder as vault** → select the cloned directory.

### 3. Install the Templater plugin

**Settings → Community plugins → Browse** → search "Templater" → Install → Enable.

Then configure it: **Settings → Templater**:
- Template folder location: `Templates`
- Trigger Templater on new file creation: off

### 4. Build the quick-capture template

Create `Templates/00-quick-capture.md` with the content from `08-deployment-guide.md § Step 5`. This is the only template required to start capturing.

Bind it to a hotkey: **Settings → Hotkeys** → search "Templater: Open Insert Template modal".

### 5. Smoke test

Run Quick Capture three times:
1. `kind: work, type: composition` — "Symphony No. 9 in D minor"
2. `kind: person, type: composer` — "Ludwig van Beethoven"
3. Open the composition note and add `composer: "[[Ludwig van Beethoven]]"` — confirm the wikilink resolves.

If it resolves, the vault is ready.

---

## Basic usage

**Capture anything** — use Quick Capture (your hotkey) to drop any item into `_Inbox/` as a stub. Never let "I'm not sure what kind this is" block a capture.

**Link in batches** — once a week, open `_Inbox/` and move notes to their folders, set `kind`/`type`, and add wikilinks to related entities. This is Stage 2 of the workflow.

**Enrich on demand** — fill in full frontmatter fields when you actually need them, not at capture time.

**Don't build Dataview queries yet** — the analysis layer is intentionally deferred until the vault has ~100+ notes. Resist.

---

## Documentation

| File | What it covers |
|---|---|
| `00-README.md` | Orientation, design principles, entity overview |
| `01-schema-core.md` | Every field on every entity type (source of truth) |
| `02-classical-edge-cases.md` | 20 patterns for classical repertoire edge cases |
| `03-onboarding-workflow.md` | Four-stage capture → enrich workflow |
| `04-templater-plan.md` | Template inventory and build order |
| `05-naming-conventions.md` | Filename patterns and disambiguation rules |
| `06-handoff-context.md` | Design rationale, open questions, known gotchas |
| `07-catalog-systems.md` | BWV, Köchel, Hoboken, and ~25 other catalog systems |
| `08-deployment-guide.md` | Full step-by-step vault setup |
| `99-schema-changelog.md` | Versioned record of schema changes |

For a new vault owner, the practical read order is: `08-deployment-guide.md` (set up) → `03-onboarding-workflow.md` (start capturing) → reference the others as needed.

---

## Schema version

Current: **v1.0.0** (2026-05-06). See `99-schema-changelog.md` for the versioning policy.
