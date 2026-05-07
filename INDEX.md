# Master Index

Quick-reference table of contents for the entire docset. Use this to find what you need without reading sequentially.

---

## Document index

| # | File | What it contains | When to read |
|---|---|---|---|
| 00 | `00-README.md` | Orientation, design principles, three-tier model, quick start | First, always |
| 01 | `01-schema-core.md` | Folder structure, base frontmatter, every entity schema (kind × type) | When defining or referencing fields |
| 02 | `02-classical-edge-cases.md` | 20 classical-music edge cases with patterns | When cataloging classical with anything beyond a single composer/work/recording |
| 03 | `03-onboarding-workflow.md` | Four-stage capture-first workflow | Before starting capture; reread when workflow drifts |
| 04 | `04-templater-plan.md` | Template inventory, prompts, build order | When building or modifying Templater templates |
| 05 | `05-naming-conventions.md` | Filename patterns, sort-title rules, disambiguation | When creating notes or renaming |
| 06 | `06-handoff-context.md` | Design rationale, decisions made, open questions, gotchas | Before making schema changes; for new agents picking up project |
| 07 | `07-catalog-systems.md` | BWV/Köchel/Hoboken/etc. — full enum with edge handling | When cataloging compositions with thematic-catalog references |
| 08 | `08-deployment-guide.md` | Step-by-step vault setup | Once, when first deploying |
| 99 | `99-schema-changelog.md` | Versioned record of schema changes | When making a schema change; consult before making one |

---

## By task

**"I want to capture a new item."**
→ `03-onboarding-workflow.md § Stage 1`. Use Quick Capture template.

**"I want to know what fields a film note should have."**
→ `01-schema-core.md § Film`.

**"I'm cataloging a Bach work."**
→ `07-catalog-systems.md § bwv`.

**"I'm cataloging a Mozart work."**
→ `07-catalog-systems.md § k`.

**"I'm cataloging a Haydn work."**
→ `07-catalog-systems.md § hob`.

**"I'm cataloging a Bartók work."**
→ `07-catalog-systems.md § Bartók's competing systems`.

**"I'm cataloging an arrangement of Mussorgsky."**
→ `02-classical-edge-cases.md § 4. Arrangements`.

**"I'm cataloging a Bruckner symphony with multiple versions."**
→ `02-classical-edge-cases.md § 5. Versions and revisions`.

**"I'm cataloging an opera recording."**
→ `02-classical-edge-cases.md § 6. Opera, oratorio, and ballet`.

**"I'm cataloging a recital album."**
→ `02-classical-edge-cases.md § 7. Recital albums and compilations`.

**"I'm cataloging a chamber music recording."**
→ `02-classical-edge-cases.md § 14. Solo and chamber recordings`.

**"I'm naming a note and want to follow the convention."**
→ `05-naming-conventions.md § Patterns by entity`.

**"Two people share a name and I need to disambiguate."**
→ `05-naming-conventions.md § Disambiguation`.

**"I want to set up the vault from scratch."**
→ `08-deployment-guide.md`.

**"I want to build a Templater template."**
→ `04-templater-plan.md` for the plan; `01-schema-core.md` for the field schemas.

**"I'm changing the schema."**
→ `06-handoff-context.md § How to evolve the schema`. Update `99-schema-changelog.md`.

**"I'm picking up this project after a hiatus."**
→ `00-README.md` then `06-handoff-context.md`.

---

## By entity

| Entity (kind / type) | Schema | Edge cases | Naming |
|---|---|---|---|
| work / composition | 01 § Composition | 02 § all | 05 § Composition |
| work / book | 01 § Book | — | 05 |
| work / film | 01 § Film | — | 05 |
| work / tv-series | 01 § TV series | — | 05 |
| work / game | 01 § Game | — | 05 |
| work / album | 01 § Album | — | 05 |
| work / article | 01 § Article | — | 05 |
| work / podcast | 01 § Podcast | — | 05 |
| work / comic | 01 § Comic | — | 05 |
| work / performance-work | 01 § Performance work | 02 § 6 | 05 |
| work / improvisation | 01 § Improvisation | 02 § 8 | — |
| recording | 01 § Recording | 02 § 6, 7, 9, 14, 16, 17, 19 | 05 § Recording |
| edition | 01 § Edition | — | 05 § Edition |
| episode | 01 § Episode | — | 05 |
| production | 01 § Production | 02 § 6 | 05 |
| collection | 01 § Collection | 02 § 10 | 05 |
| series | 01 § Series | — | 05 |
| person / composer | 01 § Composer | — | 05 § Person |
| person / author | 01 § Author | — | 05 |
| person / director | 01 § Director | — | 05 |
| person / performer | 01 § Performer | — | 05 |
| person / conductor | 01 § Conductor | — | 05 |
| person / creator | 01 § Other-creators | — | 05 |
| org / ensemble | 01 § Ensemble | — | 05 |
| org / studio | 01 § Studio | — | 05 |
| org / publisher | 01 § Publisher / Label / Network | — | 05 |
| org / label | 01 § Publisher / Label / Network | — | 05 |
| org / network | 01 § Publisher / Label / Network | — | 05 |
| place / venue | 01 § Venue | — | 05 |
| place / recording-studio | 01 § Recording studio | — | 05 |
| subject | 01 § Subject | — | 05 |

---

## Catalog system quick lookup

For composition cataloging: which system to use for which composer.

| Composer | Canonical system | Doc reference |
|---|---|---|
| J.S. Bach | bwv | `07 § bwv` |
| C.P.E. Bach | wq | `07 § Other named systems` |
| J.C. Bach | bwf | `07 § Other named systems` |
| Mozart | k (specify edition K¹–K⁹) | `07 § k` |
| Haydn | hob (group:number) | `07 § hob` |
| Beethoven | op + woo | `07 § op`, `07 § woo` |
| Schubert | d | `07 § d` |
| Schumann | op | `07 § op` |
| Mendelssohn | op or mwv | `07 § Other named systems` |
| Chopin | op or kk | `07 § Other named systems` |
| Brahms | op | `07 § op` |
| Liszt | s (Searle) | `07 § Other named systems` |
| Wagner | wwv (operas usually by title) | `07 § wwv` |
| Verdi | (titles only) | `07 § Sentinel values` |
| Tchaikovsky | op + th | `07 § Composer-to-canonical-system mapping` |
| Dvořák | op + b | `07 § Other named systems` |
| Vivaldi | rv | `07 § rv` |
| Handel | hwv | `07 § hwv` |
| Telemann | twv | `07 § Other named systems` |
| Domenico Scarlatti | kk (vs L. and P.) | `07 § kk` |
| Boccherini | g | `07 § Other named systems` |
| Bartók | sz (vs DD and BB) | `07 § Bartók's competing systems` |
| Debussy | l-debussy or CD. | `07 § Composer-to-canonical-system mapping` |
| Ravel | (none, titles only) | `07 § Sentinel values` |
| Stravinsky | (titles + year) | `07 § Sentinel values` |
| Shostakovich | op | `07 § Composer-to-canonical-system mapping` |
| Prokofiev | op | `07 § Composer-to-canonical-system mapping` |

For composers without a thematic catalog — Verdi, Ravel, most contemporary — use `catalog-system: none` with title + year + nickname disambiguation.

---

## Edge case index

Quick lookup for the 20 classical edge cases:

| # | Topic |
|---|---|
| 1 | Multi-work compositions and incomplete cycles |
| 2 | Catalog system disambiguation |
| 3 | Spurious, doubtful, fragmentary, lost works |
| 4 | Arrangements, transcriptions, orchestrations |
| 5 | Versions and revisions |
| 6 | Opera, oratorio, ballet — staging vs. recording vs. work |
| 7 | Recital albums and compilations |
| 8 | Improvisations, cadenzas, jazz/classical hybrids |
| 9 | Period instruments, tuning, pitch |
| 10 | Box sets, cycles, integral recordings |
| 11 | Multi-composer works and pasticcios |
| 12 | Anonymous, traditional, chant repertoire |
| 13 | Soundtracks and incidental music |
| 14 | Solo and chamber recordings — escaping the conductor model |
| 15 | Liturgical, sacred, functional music |
| 16 | Premiere recordings and historical firsts |
| 17 | Composer-as-performer |
| 18 | Genre and period boundaries |
| 19 | Live broadcast, radio, bootleg |
| 20 | Cross-references the schema must support |

All in `02-classical-edge-cases.md`.
