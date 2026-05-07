# Naming Conventions

Filenames are aliases — frontmatter is canonical — but consistent filenames make linking, scanning, and disambiguation dramatically easier. This document is the source of truth for filename rules.

---

## Why filename consistency matters even with frontmatter

Obsidian's `[[wikilink]]` autocomplete works on filenames. When you type `[[beet`, the dropdown shows files whose names contain "beet". If composition filenames don't include the composer, you get every Beethoven work conflated with every Beethoven recording in the suggester.

Consistent filenames also make:
- Scanning folders informative without opening each note.
- Sort-by-name produce sensible orderings.
- Disambiguation between same-titled works trivial.

---

## Patterns by entity

| Entity | Pattern | Example |
|---|---|---|
| Composition (single composer) | `Title - Composer-Last` | `Symphony No. 9 in D minor - Beethoven` |
| Composition (with catalog disambiguator) | `Title Catalog - Composer-Last` | `String Quartet No. 14 Op. 131 - Beethoven` |
| Composition (multiple composers) | `Title - Composer1 / Composer2` | `F-A-E Sonata - Schumann / Brahms / Dietrich` |
| Composition (arrangement) | `Title (Arranger orch) - OriginalComposer-Last` | `Pictures at an Exhibition (Ravel orch) - Mussorgsky` |
| Composition (version) | `Title (Year version) - Composer-Last` | `Symphony 8 (1887 version) - Bruckner` |
| Recording (single work) | `Work-short - Conductor/Performer-Last Year` | `Beethoven 9 - Karajan 1962` |
| Recording (recital) | `Performer-Last - Theme` | `Pollini - Chopin Debussy Boulez` |
| Recording (chamber/quartet) | `Work-short - Ensemble-Short Year` | `Beethoven Op 131 - Emerson 1995` |
| Book | `Title - Author-Last` | `The Master and Margarita - Bulgakov` |
| Book (translated, when translator matters) | `Title - Author-Last (Translator)` | `The Master and Margarita - Bulgakov (Pevear)` |
| Film | `Title (Year)` | `Stalker (1979)` |
| TV series | `Title (Network, Year)` | `The Wire (HBO, 2002)` |
| Game | `Title (Year)` | `Disco Elysium (2019)` |
| Album (popular music) | `Title - Artist (Year)` | `Kid A - Radiohead (2000)` |
| Article | `Title - Author-Last (Year)` | `The Crisis of the Negro Intellectual - Cruse (1967)` |
| Podcast | `Title (Network)` | `In Our Time (BBC)` |
| Comic | `Title (Publisher)` | `Sandman (Vertigo)` |
| Performance work | `Title - Author-Last` | `King Lear - Shakespeare` |
| Edition (book) | `<Work> - <Publisher Year>` | `The Master and Margarita - Penguin 1997` |
| Edition (film) | `<Work> - <Distributor Format>` | `Stalker (1979) - Criterion 4K` |
| Edition (recording) | `<Recording> - <Label Year/Format>` | `Beethoven 9 - Karajan 1962 - DG SACD 2003` |
| Episode (TV) | `<Series> S<NN>E<NN> - Title` | `The Wire S04E13 - Final Grades` |
| Episode (podcast) | `<Show> - <Date> - Title` | `In Our Time - 2020-03-12 - Albert Camus` |
| Production | `<Work> - <Venue/Year>` | `Tristan und Isolde - Bayreuth 1966` |
| Collection | descriptive | `Karajan Beethoven Cycle 1961-62` |
| Series | descriptive | `Beethoven Symphonies` |
| Person | `Full Name` | `Andrei Tarkovsky` |
| Person (disambiguated) | `Full Name (qualifier)` | `Robert Schumann (composer)` |
| Ensemble | `Common Name` | `Berlin Philharmonic` |
| Studio / Publisher / Label | `Common Name` | `Studio Ghibli` |
| Venue | `Common Name` | `Berliner Philharmonie` |
| Subject | `Title Case` | `Magical Realism` |

---

## Standard abbreviations

For recording filenames where the work portion would otherwise be unwieldy:

| Long form | Short form |
|---|---|
| Symphony No. N | Symphony N or just N |
| Piano Concerto No. N | PC N |
| Violin Concerto No. N | VC N |
| Cello Concerto No. N | CC N |
| String Quartet No. N | SQ N |
| Piano Sonata No. N | PS N |
| Op. NNN | Op NNN |
| BWV NNN | BWV NNN (no shortening) |
| K. NNN | K NNN |

So:
- `Symphony No. 9 in D minor - Beethoven` (composition full)
- `Beethoven 9 - Karajan 1962` (recording short — "Beethoven 9" is unambiguous)
- `Beethoven SQ 14 - Emerson 1995` (recording short for less famous works)

When in doubt, longer is safer. Filenames can always be aliased via the `aliases:` array in frontmatter, so old links don't break.

---

## Sort-title rules

The `sort-title:` field in frontmatter normalizes for sorting purposes. Filename can keep the natural form.

**Rules:**

1. Strip leading articles in the source language: *The*, *A*, *An* (English); *Le*, *La*, *Les*, *L'* (French); *Der*, *Die*, *Das* (German); *El*, *La*, *Los*, *Las* (Spanish); *Il*, *La*, *Lo*, *Gli*, *Le* (Italian).
   - `The Wire` → sort-title: `Wire, The`
   - `Le Sacre du printemps` → sort-title: `Sacre du printemps, Le`
   - `Der Ring des Nibelungen` → sort-title: `Ring des Nibelungen, Der`

2. For numbered works, zero-pad to allow lexicographic sort:
   - `Symphony No. 9 in D minor` → sort-title: `Symphony No. 09 in D minor`
   - This matters for catalogs reaching double or triple digits (Bach BWV 1080, Haydn 104 symphonies).

3. For composer-prefixed sort orderings (people), use `Last, First`:
   - `Ludwig van Beethoven` → sort-name: `Beethoven, Ludwig van`
   - `Johann Sebastian Bach` → sort-name: `Bach, Johann Sebastian`
   - `Jean-Philippe Rameau` → sort-name: `Rameau, Jean-Philippe`

4. Surname conventions:
   - "von", "van", "de", "della" — lowercase; surname starts at the next capitalized word.
     - `Ludwig van Beethoven` → `Beethoven, Ludwig van`
     - `Heitor Villa-Lobos` → `Villa-Lobos, Heitor` (hyphenated is one surname)
   - Spanish/Portuguese double surnames — preserve both, sort by the first.
     - `Joaquín Rodrigo Vidre` → `Rodrigo Vidre, Joaquín`

---

## Disambiguation

When two notes would otherwise collide on filename:

1. **Same-name people** — append discipline or dates in parentheses:
   - `Robert Schumann (composer)`
   - `Robert Schumann (psychiatrist)`
   - `Johann Strauss I` / `Johann Strauss II` (numerals when they're the canonical form)

2. **Same-title works by different composers** — composer in filename per the standard pattern handles this:
   - `Requiem - Mozart`
   - `Requiem - Verdi`
   - `Requiem - Brahms` (technically *Ein deutsches Requiem* — use the formal title)

3. **Same-title films from different years** — year suffix per standard pattern:
   - `Solaris (1972)`
   - `Solaris (2002)`

4. **Same composer, multiple works with same generic title** — catalog disambiguator:
   - `String Quartet No. 14 Op. 131 - Beethoven`
   - `String Quartet No. 14 Op. 18 No. 4 - Beethoven` (would never collide given Op. but illustrates the rule)
   - For Schubert quartets: `String Quartet No. 14 D. 810 (Death and the Maiden) - Schubert`

5. **Recordings of same work, same conductor, different year** — year disambiguates:
   - `Beethoven 9 - Karajan 1962`
   - `Beethoven 9 - Karajan 1977`
   - `Beethoven 9 - Karajan 1982`

6. **Recordings of same work, same conductor, same year, different sources** — append source:
   - `Beethoven 9 - Furtwängler 1942 (Berlin)`
   - `Beethoven 9 - Furtwängler 1942 (Stockholm)`

---

## Aliases

The frontmatter `aliases:` array supports finding notes via wikilinks even when the filename doesn't match. Use aggressively:

```yaml
aliases:
  - "Choral Symphony"
  - "Beethoven 9th"
  - "Beethoven's Ninth"
  - "Symphony No. 9"
  - "Symphony 9"
  - "Op. 125"
  - "BWV-equivalent... "         # for cross-catalog lookup
  - "Symphony No. 9 in D minor, Op. 125"
```

Aliases work in:
- Wikilink autocomplete (`[[Choral` resolves to the file).
- Quick switcher.
- Search.

Original-language titles always go in `aliases:`:

```yaml
title: "The Master and Margarita"
original-title: "Мастер и Маргарита"
aliases:
  - "Мастер и Маргарита"
  - "Master i Margarita"
```

---

## Renaming protocol

If you rename a file:

1. Use Obsidian's built-in rename (F2 or right-click → Rename). It updates all `[[wikilinks]]` automatically.
2. Add the *old* filename to `aliases:`. This catches stale references in unstructured prose, external bookmarks, etc.
3. If the rename was due to a schema convention change, log it in a future `99-schema-changelog.md`.

Don't rename casually. Filenames stabilize the graph; churn destabilizes it. Most "should I rename" cases resolve toward "no, just add an alias."

---

## Special characters

Avoid in filenames (Obsidian / OS / sync conflicts):

- `:` — replace with `-` or omit
- `/` — replace with `-` or omit  
- `\` — never
- `|` — never
- `?` — never
- `*` — never
- `"` — replace with `'` or omit
- `<` `>` — never

OK in filenames:
- `,` `.` `(` `)` `[` `]` `'` `-` `&`
- Unicode letters (accents, non-Latin scripts) — work but may cause sync issues across some platforms; default to transliterated forms in the filename and put the original in `aliases:` and `original-title:`.

Examples of necessary transformation:

- `8½` → `8 1-2 (1963)` (filename), original `8½` in title
- `WALL·E` → `WALL-E (2008)` (filename)
- `Бгу: ...` (Cyrillic) → transliterated form in filename, Cyrillic in `aliases:` and `original-title:`
