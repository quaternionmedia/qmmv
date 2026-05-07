# Catalog Systems Reference

A reference for the major thematic-catalog systems used to identify classical compositions, with the schema patterns for each. Bach (BWV) and Mozart (Köchel) are the most structurally complex and drive the design choices below; everything else slots into the same model.

This document is the authoritative source for `catalog-system` values and `catalog` field formatting. The composer note declares which systems are canonical for that composer; the composition note records the actual catalog reference using the patterns here.

---

## The structural challenge

Different catalog systems have different shapes:

- **Linear-numeric** (Köchel K., Deutsch D., Schubert pre-Deutsch Op., Ryom RV) — a single integer.
- **Linear-numeric with variant suffixes** (BWV 772a, K. 45a) — integer plus single-letter variant.
- **Linear-numeric with sub-version dots** (BWV 205.1, BWV 208.1 in BWV³) — integer plus dotted sub-version, introduced by the 2022 third edition.
- **Group-and-number** (Hoboken `Hob. I:101`) — Roman numeral group, then colon, then arabic numeral within group.
- **Multi-section with appendix** (BWV main + BWV Anh. I/II/III) — main catalog plus separate Anhang for doubtful/spurious/lost works.
- **Edition-dependent** (Köchel K. vs. K⁶; Bartók Sz. vs. DD vs. BB) — the same work may have different numbers in different scholarly editions.
- **Composite display** (Schubert "Op. 90, D. 899" — both shown together) — multiple systems referenced simultaneously.

The schema accommodates all of these via three fields: `catalog` (the canonical reference as a display string), `catalog-system` (an enum tag), and `catalog-alternates:` (an array of additional system/number pairs).

---

## Schema fields (canonical)

```yaml
# On the composition note:
catalog: "BWV 1080"               # display form of the canonical reference
catalog-system: bwv               # enum tag for the system; controls sorting and dispatch
catalog-number: 1080              # numeric portion only, for sorting
catalog-suffix: ""                # variant letter ("a", "b") if applicable
catalog-group: ""                 # for group-and-number systems (Hoboken: "I", "XVI")
catalog-section: ""               # for systems with appendices ("Anh. I", "Anh. II", "Anh. III")
catalog-edition: null             # which edition of the catalog ("BWV3", "K6", "Sz")
catalog-alternates:               # other catalog systems referencing this same work
  - { system: bc,    number: "B 28" }
  - { system: op,    number: "Op. posth." }
  - { system: bg,    number: "BG 25/2" }

# On the composer note:
catalog-systems-canonical: [bwv]      # the system this composer's works are typically referenced by
catalog-systems-alternate: [bc, op, bg, anh]   # other systems users may encounter
catalog-edition-canonical: BWV3       # which edition the canonical references use, if multiple exist
```

**Sort order.** Always sort by `(catalog-system, catalog-group, catalog-number, catalog-suffix)`. The `catalog-number` field exists specifically so sorting works numerically without parsing the display string.

**Filename.** Use the display form of `catalog` when disambiguation is needed: `String Quartet No. 14 Op. 131 - Beethoven`, `Variations BWV 988 - Bach`, `Symphony Hob. I:101 - Haydn`. Otherwise, work title alone is fine.

---

## The `catalog-system` enum (full list)

Each entry below documents: the abbreviation, the composer(s) it covers, the cataloger, the format pattern, and any structural quirks the schema must handle.

### Mainstream systems

#### `bwv` — Bach-Werke-Verzeichnis

- **Composer:** Johann Sebastian Bach
- **Cataloger:** Wolfgang Schmieder, 1950
- **Editions:** BWV (1950), BWV² (1990), BWV²a (1998), BWV³ (2022)
- **Format:** `BWV NNNN` for main catalog; `BWV Anh. I/II/III NNN` for the three appendix sections
- **Quirks:**
  - Some catalogues have appendices (German: Anhang, abbreviated as Anh.) for doubtful and/or spurious works, arrangements, etc.
  - The 1990 second edition reclassified over 20 works as spurious, moving them from the main catalogue to the appendix (Anhang), and expanded the BWV Anh. to 213 entries.
  - Variant indicators use an "a" suffix for revised versions (e.g., BWV 772a for an early variant of the Invention in C major).
  - BWV³ (2022) introduced dotted sub-version numbers — BWV 205.1 (previously 205) and BWV 205.2 (previously 205a) — replacing some older suffix conventions.
  - Numbers above BWV 1126 were added in the 21st century.
- **Schema pattern:**

```yaml
# Standard work
catalog: "BWV 1080"
catalog-system: bwv
catalog-number: 1080
catalog-edition: BWV3

# Variant
catalog: "BWV 772a"
catalog-system: bwv
catalog-number: 772
catalog-suffix: "a"

# Sub-version (BWV3)
catalog: "BWV 205.1"
catalog-system: bwv
catalog-number: 205
catalog-suffix: ".1"
catalog-edition: BWV3

# Anhang
catalog: "BWV Anh. II 80"
catalog-system: bwv
catalog-section: "Anh. II"
catalog-number: 80
attribution-status: doubtful   # nearly always for Anh. II/III
```

#### `bc` — Bach Compendium (alternate to BWV)

- **Composer:** Johann Sebastian Bach
- **Cataloger:** Hans-Joachim Schulze and Christoph Wolff, 1985–
- **Format:** `BC X NN` where X is a section letter (A vocal, B chorales, C concerted, etc.) and NN is the number within the section
- **Use:** As an alternate; BWV remains canonical. Useful when a work has no BWV number but has a BC number.
- **Schema pattern:** Use as `catalog-alternates:` entry, or as the canonical for works that exist in BC but not BWV.

#### `k` — Köchel-Verzeichnis

- **Composer:** Wolfgang Amadeus Mozart
- **Cataloger:** Ludwig Ritter von Köchel, 1862
- **Editions:** K¹ (1862) through K⁹ (most recent, edited Neal Zaslaw); K⁶ (1964) is widely used
- **Format:** `K. NNN` or `KV NNN` (German usage)
- **Quirks:**
  - Catalogs can be revised by later scholars, notably the nine editions of the standard Mozart catalog, created by a series of editors stretching from the founding editor Köchel to (most recently) Neal Zaslaw.
  - Modern scholarship and the discovery of lost manuscripts have made some revisions to the list, which is why you'll see things like Mozart: Symphony in G, "Alte Lambach", K. 45a (the "a" denotes an insertion between K. 45 and K. 46 of the original list).
  - Many works have different numbers in K¹ vs K⁶ vs K⁹. Recordings and concert programs use K. or K⁶ inconsistently.
  - K⁶ (Sechste Auflage, 1964) is the most commonly cited modern edition.
- **Schema pattern:**

```yaml
catalog: "K. 626"
catalog-system: k
catalog-number: 626
catalog-edition: K6
catalog-alternates:
  - { system: k, number: "K¹ 626", edition: K1 }   # if same number across editions, no alternate needed
```

For K⁶-vs-original-K differences, prefer K⁶ as canonical and put the original K¹ number in `catalog-alternates`.

#### `d` — Deutsch-Verzeichnis

- **Composer:** Franz Schubert
- **Cataloger:** Otto Erich Deutsch, 1951
- **Format:** `D. NNN` or `D NNN`
- **Quirks:** In some cases, both the opus number and the newer catalogue designation are appended to a work. For example, Schubert's first set of Impromptus was published as Op. 90 and is now catalogued as D 899, but concert programmes, CDs and reference works commonly refer to Schubert's "Impromptus, Op. 90, D. 899".
- **Schema pattern:**

```yaml
catalog: "D. 899"
catalog-system: d
catalog-number: 899
catalog-alternates:
  - { system: op, number: "Op. 90" }
```

#### `hob` — Hoboken-Verzeichnis

- **Composer:** Joseph Haydn
- **Cataloger:** Anthony van Hoboken, 1957–1978
- **Format:** `Hob. <Roman>:<Arabic>` (e.g., `Hob. I:101`, `Hob. XVI:6`)
- **Quirks:**
  - Unlike other catalogues, he employed Roman numerals at the beginning. Symphonies start off with the numeral I; string quartets start off with the numeral III; concertos start off with the numeral VII.
  - Group structure: I = symphonies, II = divertimenti (4+ parts), III = string quartets, IV = trios, V = string trios, VI = duos, VII = concertos (further subdivided VIIa-VIIh by instrument), VIII = marches, IX = dances, X = baryton works (subdivided), XI–XII = lira works, XIII = baryton octets, XIV = various trios, XV = piano trios, XVa = violin sonatas, XVI = piano sonatas, XVII = piano works, XVIII = piano concertos, XIX = mechanical clock works, XX = Seven Last Words, XXI–XXIV = vocal/sacred, XXV–XXIX = various vocal, XXX–XXXII = miscellaneous.
  - Haydn opus numbers also commonly used for string quartets — In the case of Joseph Haydn's string quartets, the opus numbers continue to be used, to some advantage: Haydn mostly wrote his quartets in sets of six, which were published under a single opus number (e.g., Opus 33, no. 1, no. 2 ... no. 6).
  - Landon (L.) numbers exist for piano sonatas as alternate to Hob. XVI.
- **Schema pattern:**

```yaml
catalog: "Hob. I:101"
catalog-system: hob
catalog-group: "I"
catalog-number: 101
nickname: "The Clock"

# String quartet — both Hob. and Op.
catalog: "Hob. III:77"
catalog-system: hob
catalog-group: "III"
catalog-number: 77
catalog-alternates:
  - { system: op, number: "Op. 76 No. 3" }
nickname: "Emperor"

# Piano sonata with Landon alternate
catalog: "Hob. XVI:6"
catalog-system: hob
catalog-group: "XVI"
catalog-number: 6
catalog-alternates:
  - { system: l-haydn, number: "L. 13" }
```

Note `l-haydn` to disambiguate from Scarlatti's L. (Longo).

#### `op` — Opus number

- **Composer:** Many (Beethoven, Brahms, Chopin, Schumann, Mendelssohn, Dvořák, Tchaikovsky, etc.)
- **Cataloger:** N/A — opus numbers are typically assigned by publishers in publication order
- **Format:** `Op. NN` or `Op. NN No. M` or `Op. NN/M`
- **Quirks:** Works in collections (Beethoven Op. 18 string quartets, Op. 59 "Razumovsky") use `Op. NN No. M` or `Op. NN/M`. Posthumous works: `Op. posth.` or with a number.
- **Schema pattern:**

```yaml
catalog: "Op. 131"
catalog-system: op
catalog-number: 131

# Within-set
catalog: "Op. 18 No. 4"
catalog-system: op
catalog-number: 18
catalog-suffix: " No. 4"        # preserved as part of display
# Or use a sub-number field if you want clean numeric sorting:
catalog-sub-number: 4
```

#### `woo` — Werke ohne Opuszahl

- **Composer:** Beethoven primarily (also others)
- **Cataloger:** Georg Kinsky and Hans Halm, 1955 (the "Kinsky-Halm Catalogue")
- **Format:** `WoO NN`
- **Use:** For Beethoven works that never received an opus number because they were never published.
- **Schema pattern:** Treat as its own system; same composer's note declares both `op` and `woo` as canonical.

```yaml
catalog: "WoO 59"
catalog-system: woo
catalog-number: 59
nickname: "Für Elise"
```

Hess (Willy Hess) numbers are the alternate Beethoven catalog for unpublished works; use `hess` if needed in `catalog-alternates`.

#### `rv` — Ryom-Verzeichnis

- **Composer:** Antonio Vivaldi
- **Cataloger:** Peter Ryom, 1970s; revised 2007
- **Format:** `RV NNN`
- **Schema pattern:** Linear-numeric. Straightforward.

```yaml
catalog: "RV 269"
catalog-system: rv
catalog-number: 269
nickname: "La primavera (Spring)"
```

#### `hwv` — Händel-Werke-Verzeichnis

- **Composer:** George Frideric Handel
- **Cataloger:** Bernd Baselt, 1978–1986
- **Format:** `HWV NNN`
- **Schema pattern:** Linear-numeric. Their usage isn't yet as widespread as the BWV numbers — opus numbers and titles still common.

```yaml
catalog: "HWV 351"
catalog-system: hwv
catalog-number: 351
title: "Music for the Royal Fireworks"
```

#### `wwv` — Wagner-Werk-Verzeichnis

- **Composer:** Richard Wagner
- **Cataloger:** John Deathridge, Martin Geck, Egon Voss, 1986
- **Format:** `WWV NN`
- **Use:** Less commonly cited than work titles for major works — operas are referenced by title.

#### `kk` (or `k-scarlatti`) — Kirkpatrick

- **Composer:** Domenico Scarlatti
- **Cataloger:** Ralph Kirkpatrick, 1953
- **Format:** `K. NNN` (creates collision with Mozart) or `Kk NNN` (disambiguated)
- **Quirks:** These pieces are also given a K number, usually written "Kk" to distinguish it from Mozart's Köchel number. Three competing Scarlatti systems exist:
  - **L.** (Longo, early 20th c.) — L. NNN
  - **Kk** (Kirkpatrick, 1953) — Kk NNN
  - **P.** (Pestelli, 1967) — P. NNN
  - The Kk and Longo numbers do not correspond, which can make identifying a particular work by Scarlatti tricky.
- **Schema pattern:** Use `kk` as canonical (most common in modern scholarship); record L. and P. in `catalog-alternates`.

```yaml
catalog: "Kk 141"
catalog-system: kk
catalog-number: 141
catalog-alternates:
  - { system: l-scarlatti, number: "L. 422" }
  - { system: p, number: "P. 271" }
```

### Bartók's competing systems

Three attempts—two full and one partial—have been made at cataloguing. The first, and still most widely used, is András Szőllősy's chronological Sz numbers, from 1 to 121. Denijs Dille subsequently reorganised the juvenilia (Sz. 1–25) thematically, as DD numbers 1 to 77. The most recent catalogue is that of László Somfai; this is a chronological index with works identified by BB numbers 1 to 129.

- `sz` — Szőllősy (canonical, most widely cited)
- `dd` — Dille (juvenilia only)
- `bb` — Somfai (most recent, increasingly used)

```yaml
catalog: "Sz. 106"
catalog-system: sz
catalog-number: 106
catalog-alternates:
  - { system: bb, number: "BB 115" }
title: "Music for Strings, Percussion and Celesta"
```

### Other named systems

| System | Composer | Cataloger | Notes |
|---|---|---|---|
| `bwv` `bc` | J.S. Bach | Schmieder / Schulze-Wolff | See above |
| `bwv-cpe` `wq` `h` (Helm) | C.P.E. Bach | Wotquenne / Helm | Wq and H both used |
| `bwv-jc` `bwf` | J.C. Bach (London Bach) | Warburton | |
| `fk` `f` | W.F. Bach | Falck | |
| `bv` | Busoni | Kindermann | Busoni-Verzeichnis |
| `kk` (or `k-chopin`) | Chopin | Kobylańska | KK; works without opus |
| `b` | Dvořák | Burghauser | B. NNN; canonical for Dvořák alongside Op. |
| `cd` | Debussy | Lesure | L. NNN (yet another L.); also CD. |
| `s` | Liszt | Searle | S. NNN |
| `m` | Mendelssohn | MWV (Mendelssohn-Werkverzeichnis) | MWV |
| `tk` | Tchaikovsky | TH | TH NNN |
| `tcw` `r` | Rachmaninoff | TN (Threlfall-Norris) | |
| `s-saint-saens` `r` | Saint-Saëns | Ratner | R. NNN |
| `g` | Boccherini | Gérard | G. NNN |
| `p` `kk` | Pergolesi | Paymer | |
| `zwv` | Zelenka | Zelenka-Werkverzeichnis | ZWV |
| `tt` `twv` | Telemann | Telemann-Werkverzeichnis | TWV |
| `bb` `kov` | Khachaturian | Yuzefovich | |
| `op` `op-posth` | many | publishers | see above |
| `woo` | Beethoven (primary), others | Kinsky-Halm | see above |
| `anh` | many | Anhang of various catalogs | doubtful/spurious; usually paired with parent system |
| `bg` | J.S. Bach | Bach-Gesellschaft (1851–1899) | BG NN; precursor to BWV |
| `nba` | J.S. Bach | Neue Bach-Ausgabe | NBA series/volume; for editions, not works |

When you encounter a system not in this list, add it as a row to this table when you next edit the docs. Don't put novel systems in `catalog-system` without documenting them here — Dataview queries that group by system will silently miss undocumented values.

### Sentinel values

| Value | Use |
|---|---|
| `none` | Work has no catalog reference at all (recent commission, anonymous, traditional) |
| `other` | Unusual or local catalog system not worth promoting to the enum |
| `op` only | Pre-catalog-system composers who are still referenced by Op. (most 19th-century) |

---

## Composer-to-canonical-system mapping (quick reference)

For populating `catalog-systems-canonical:` on composer notes:

| Composer | Canonical | Common alternates |
|---|---|---|
| J.S. Bach | bwv | bc, bg, op, anh |
| C.P.E. Bach | wq | h |
| J.C. Bach | bwf | op |
| W.F. Bach | fk | |
| Mozart | k | (K¹ vs K⁶ via catalog-edition) |
| Haydn | hob | op (for quartets), l-haydn (for piano sonatas) |
| Beethoven | op, woo | hess, anh |
| Schubert | d | op |
| Mendelssohn | op | mwv |
| Schumann | op | woo |
| Brahms | op | woo |
| Chopin | op | kk-chopin |
| Liszt | s | op |
| Wagner | wwv | (operas almost always referenced by title) |
| Verdi | (title) | |
| Tchaikovsky | op | th |
| Dvořák | op, b | |
| Vivaldi | rv | op |
| Handel | hwv | op |
| Telemann | twv | |
| Domenico Scarlatti | kk | l-scarlatti, p |
| Boccherini | g | op |
| Bartók | sz | bb, dd |
| Debussy | l-debussy (sometimes CD.) | op |
| Ravel | (none, just titles, sometimes M.) | |
| Stravinsky | (titles + year) | |
| Shostakovich | op | |
| Prokofiev | op | |

For composers without a thematic catalog (Verdi operas, Ravel, most contemporary), `catalog-system: none` and rely on title + year + nickname.

---

## Filename pattern reminders

Recall from `05-naming-conventions.md` — when catalog disambiguation is in the filename, use the canonical display form:

- `Symphony No. 9 in D minor - Beethoven` (no catalog needed; nickname unique)
- `String Quartet No. 14 Op. 131 - Beethoven` (catalog needed; multiple Op. 131 candidates if abbreviated)
- `Goldberg Variations BWV 988 - Bach` (BWV included for searchability)
- `Symphony Hob. I:101 (The Clock) - Haydn` (full Hob. reference; Roman/Arabic preserved)
- `Sonata K. 141 - Scarlatti` (note: this is Kk; "K." common in informal use, but in vault prefer `Kk` to avoid Mozart-Scarlatti collision)
- `Death and the Maiden D. 810 - Schubert`
- `Music for Strings, Percussion and Celesta Sz. 106 - Bartók`

For colon-bearing Hoboken numbers, use the format directly — Obsidian filenames accept `:` on Linux/Mac but not Windows. If cross-platform sync matters, substitute `Hob I-101` and put the canonical `Hob. I:101` in frontmatter and `aliases:`.

---

## What this means for the schema as already written

The base composition schema in `01-schema-core.md` already has the right fields, but the `catalog-system` enum was abbreviated. The full enum is defined here. Composer notes should declare both `catalog-systems-canonical:` and `catalog-systems-alternate:` to enable correct dispatch when you encounter ambiguous references in source material.

The Templater composition template should:

1. After capturing the composer link, look up the composer's `catalog-systems-canonical:` and prefill `catalog-system:`.
2. For Hoboken (`hob`), prompt for group and number separately.
3. For BWV, prompt for section (main / Anh. I / II / III) and number, with optional variant suffix.
4. For Köchel, prompt for which edition (K¹, K⁶, K⁹) the number references.
5. For Bartók, prompt which of the three systems (Sz / DD / BB) is being used.

These prompts run only when the composer has been resolved — for stub captures, leave `catalog:` as a free string and refine later.
