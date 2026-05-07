# Classical Music Edge Cases

Classical music is the stress test for any media-cataloging schema. The Work/Recording/Edition split handles the common case — Beethoven's 9th, Karajan/BPO 1962, the 2003 SACD remaster — but the repertoire generates dozens of structural exceptions that need defined handling. Each section below identifies the case, the schema decision, and the pattern.

---

## 1. Multi-work compositions and incomplete cycles

**Case.** A Composition can be a single work (Symphony No. 5) or a *cycle* whose parts are themselves substantial works (the Goldberg Variations as 30 variations + aria; *Das Wohltemperierte Klavier* Book I as 24 prelude/fugue pairs; *Winterreise* as 24 Lieder). Sometimes you want to talk about the cycle, sometimes about an individual part.

**Decision.** Treat the cycle as the canonical Composition. Promote individual parts to their own Composition notes only when (a) they are routinely performed standalone, (b) you have recordings of just that part, or (c) you want to write about it.

**Pattern.**

```yaml
# /Works/Compositions/Goldberg Variations - Bach.md
kind: work
type: composition
title: "Goldberg Variations"
catalog: "BWV 988"
catalog-system: bwv
composer: "[[Johann Sebastian Bach]]"
form: theme-and-variations
sub-form: aria-with-variations
parts-count: 32           # aria + 30 variations + aria da capo
parts-summary: |
  Aria, 30 variations grouped in canons (every 3rd), 
  genre pieces, and virtuoso showpieces; aria da capo.
has-named-parts: true
parts-promoted: ["[[Goldberg Variations - Variation 25 - Bach]]"]   # only if you've broken one out
```

Use `parts-promoted` to track which sub-works have their own notes. Avoid creating 30 stubs preemptively.

**Counter-case (don't split).** *St Matthew Passion* arias are not normally treated as standalone works — the whole work is the unit. Use `movements:` array (the existing field) rather than promoting movements to separate notes.

---

## 2. Catalog system disambiguation

**Case.** A composer may have multiple overlapping catalog systems with structurally different formats. Bach has BWV (Schmieder, with Anh. for spurious/lost), BC (Bach Compendium), and BG (Bach-Gesellschaft). The BWV itself has four editions (1950, 1990, 1998, 2022) that re-number some works (e.g. BWV 205.1 in BWV³ replaces older BWV 205). Mozart has K¹ through K⁹ — same work, different numbers across editions. Haydn has Hob. (group:number format like Hob. I:101) *and* Op. for string quartets *and* Landon (L.) for piano sonatas. Bartók has three competing catalogs (Sz. / DD. / BB.). Scarlatti has three competing catalogs (Kk. / L. / P.). Beethoven has Op. *and* WoO *and* Hess.

**Decision.** Treat catalog metadata as a structured data problem, not a string. Three fields capture the canonical reference (`catalog`, `catalog-system`, `catalog-number` for sortability); additional fields handle structural variation (`catalog-group` for Hoboken, `catalog-section` for Bach Anh., `catalog-suffix` for variants, `catalog-edition` for Köchel/BWV editions); `catalog-alternates:` carries other-system references.

**Authoritative reference.** See `07-catalog-systems.md` for the full enum of catalog systems, format patterns per system, the BWV-edition handling, the Köchel-edition handling, the Hoboken group structure, and the composer-to-canonical-system mapping.

**Quick patterns:**

```yaml
# composer note
catalog-systems-canonical: [bwv]
catalog-systems-alternate: [bc, bg, op, anh]
catalog-edition-canonical: BWV3

# straightforward composition
catalog: "BWV 1080"
catalog-system: bwv
catalog-number: 1080
catalog-edition: BWV3

# Hoboken group:number
catalog: "Hob. I:101"
catalog-system: hob
catalog-group: "I"
catalog-number: 101

# BWV with variant suffix
catalog: "BWV 772a"
catalog-system: bwv
catalog-number: 772
catalog-suffix: "a"

# BWV3 dotted sub-version
catalog: "BWV 205.1"
catalog-system: bwv
catalog-number: 205
catalog-suffix: ".1"
catalog-edition: BWV3

# Bach Anhang (almost always doubtful/spurious)
catalog: "BWV Anh. II 80"
catalog-system: bwv
catalog-section: "Anh. II"
catalog-number: 80
attribution-status: doubtful

# Köchel — note which edition
catalog: "K. 626"
catalog-system: k
catalog-number: 626
catalog-edition: K6

# Multi-system display (Schubert with both D. and Op.)
catalog: "D. 899"
catalog-system: d
catalog-number: 899
catalog-alternates:
  - { system: op, number: "Op. 90" }
```

**Sort rule.** Sort by `(catalog-system, catalog-group, catalog-number, catalog-suffix)`. The `catalog-number` field exists specifically so sorting works numerically without parsing display strings. Filenames use the canonical display form (`Goldberg Variations BWV 988 - Bach`, `Symphony Hob. I:101 - Haydn`).

---

## 3. Spurious, doubtful, fragmentary, and lost works

**Case.** Bach's *Toccata and Fugue in D minor* BWV 565 is now widely considered spurious. Mozart's K. 297b *Sinfonia Concertante* survives only in a problematic source. Many Renaissance works are anonymous or fragmentary. Beethoven sketched a 10th Symphony.

**Decision.** Add an `attribution-status` field to Composition. Treat all of these as legitimate notes — scholarship and recording history matter even when the work itself is contested.

**Pattern.**

```yaml
attribution-status: confirmed | doubtful | spurious | anonymous | fragmentary | lost | reconstructed
attributed-to: "[[Johann Sebastian Bach]]"     # use this when attribution-status != confirmed
attributed-also-to: ["[[Johann Peter Kellner]]"]
completion-by: "[[Barry Cooper]]"              # for reconstructions
sources-status: "Single 19th-c manuscript copy"
```

Filename for spurious works: keep the traditional attribution since that's how you'll search — `Toccata and Fugue in D minor BWV 565 - Bach`.

---

## 4. Arrangements, transcriptions, and orchestrations

**Case.** Ravel's orchestration of *Pictures at an Exhibition*. Liszt's piano transcriptions of Beethoven symphonies. Bach's keyboard arrangements of Vivaldi concertos. Stokowski's Bach orchestrations. Schoenberg's chamber arrangement of Mahler 4. Berio's *Rendering* (Schubert symphonic sketches).

**Decision.** Arrangements are *derivative works* — separate Composition notes that link to the source. Don't try to fold them into the original composition.

**Pattern.**

```yaml
# /Works/Compositions/Pictures at an Exhibition (Ravel orch) - Mussorgsky.md
kind: work
type: composition
title: "Pictures at an Exhibition (Ravel orchestration)"
sort-title: "Pictures at an Exhibition, Ravel orchestration"
composer: "[[Modest Mussorgsky]]"      # original composer
arranger: "[[Maurice Ravel]]"
arrangement-of: "[[Pictures at an Exhibition - Mussorgsky]]"
arrangement-type: orchestration        # orchestration | transcription | reduction | reharmonization | completion | reconstruction | paraphrase
arrangement-year: 1922
instrumentation: [orchestra]           # the new instrumentation, not the original's
notes-arrangement: |
  Commissioned by Koussevitzky. Bypasses the "Bydło" promenade; 
  changes orchestral color of "Bydło" itself.
```

The original composition note gains an `arrangements` array pointing to all known arrangements. Recordings link to whichever Composition was actually performed.

**Cadenzas** are a sub-case: Brahms's cadenza for Mozart's D-minor concerto K. 466. Either treat as a tiny derivative Composition, or as a `cadenzas:` sub-list on the parent concerto note (with composer and link). The latter is usually cleaner.

---

## 5. Versions, revisions, and editions of works

**Case.** Bruckner's symphonies in their multiple versions (Symphony 8: 1887 / 1890, Nowak / Haas editions). Sibelius's *5th Symphony* (1915 / 1916 / 1919). Mussorgsky's *Boris Godunov* (1869 / 1872 / Rimsky / Shostakovich). Stravinsky's *Petrushka* (1911 / 1947).

This is distinct from arrangements: it's the same composer revising their own work, sometimes decades apart, producing meaningfully different scores.

**Decision.** Composition note represents the work *abstractly*. Versions are tracked as a structured array on the work, *and* promoted to their own Composition notes when distinct enough that recordings need to specify which one.

**Pattern.**

```yaml
# /Works/Compositions/Symphony No. 8 in C minor - Bruckner.md
kind: work
type: composition
title: "Symphony No. 8 in C minor"
composer: "[[Anton Bruckner]]"
catalog: "WAB 108"
versions:
  - { id: 1887, year: 1887, name: "First version", editor: null,
      promoted: "[[Symphony 8 (1887 version) - Bruckner]]" }
  - { id: 1890, year: 1890, name: "Second version (revised)", editor: null }
  - { id: nowak, year: 1955, name: "Nowak edition (of 1890)", 
      editor: "[[Leopold Nowak]]" }
  - { id: haas, year: 1939, name: "Haas edition (conflated)", 
      editor: "[[Robert Haas]]" }
default-version: 1890
```

Recording schema gains a `version:` field referencing the version id:

```yaml
# recording
work: "[[Symphony No. 8 in C minor - Bruckner]]"
version: 1887                          # or "nowak", "haas", etc.
version-note: "Inbal's pioneering 1887 recording"
```

For editorial editions specifically (Nowak vs. Haas vs. Urtext), `score-edition` can also be tracked at recording level when relevant.

---

## 6. Opera, oratorio, and ballet — staging vs. recording vs. work

**Case.** *Tristan und Isolde* the work, vs. the Bayreuth 1966 production directed by Wieland Wagner, vs. the audio-only recording made of that production, vs. the DVD release of a different staging.

**Decision.** Three layers, all entities:
- The **Performance Work** (the opera itself — text + music).
- The **Production** — a specific staging (director, designer, run dates, venue).
- The **Recording** — audio or video, of either a production or studio.

Productions are an additional `kind` worth introducing for staged works. Most users don't need them; introduce only when you have multiple recordings of the same production or want to write about staging itself.

**Pattern.**

```yaml
# /Works/Performances/Tristan und Isolde - Wagner.md
kind: work
type: performance-work
sub-type: opera
composer: "[[Richard Wagner]]"
librettist: "[[Richard Wagner]]"
acts: 3
duration-typical: 240
language: German
premiered: 1865-06-10

# /Productions/Tristan und Isolde - Bayreuth 1966.md   (optional)
kind: production
type: production
of: "[[Tristan und Isolde - Wagner]]"
director: "[[Wieland Wagner]]"
set-designer: "[[Wieland Wagner]]"
costume-designer: null
conductor: "[[Karl Böhm]]"
premiere-date: 1966-07-25
venue: "[[Bayreuth Festspielhaus]]"
run-end: 1970
notes-production: "Late Wieland minimalism, abstract circular set."

# /Recordings/Tristan und Isolde - Böhm Bayreuth 1966.md
kind: recording
type: recording
work: "[[Tristan und Isolde - Wagner]]"
production: "[[Tristan und Isolde - Bayreuth 1966]]"
recording-medium: audio                # audio | video
conductor: "[[Karl Böhm]]"
ensemble: "[[Bayreuth Festival Orchestra]]"
choir: "[[Bayreuth Festival Chorus]]"
soloists:
  - { performer: "[[Birgit Nilsson]]", role: "Isolde" }
  - { performer: "[[Wolfgang Windgassen]]", role: "Tristan" }
recording-type: live
recording-venue: "[[Bayreuth Festspielhaus]]"
recorded-start: 1966-07-25
```

For ballet, add `choreographer:` and `dancers:` to the production note.

---

## 7. Recital albums and compilation recordings

**Case.** A Pollini recital with Chopin, Debussy, and Boulez. A Hilary Hahn album titled *Bach: Violin Concertos* with three works. A "best of" compilation. A historic recording reissue spanning a career.

**Decision.** The Recording note's `works:` array (plural) is the primary mechanism. Each entry can be a simple link or a richer object with track-level metadata.

**Pattern — simple recital.**

```yaml
kind: recording
title: "Pollini - Chopin / Debussy / Boulez"
work: null                           # explicitly null when multi-work
works:
  - { work: "[[Préludes Op. 28 - Chopin]]", tracks: "1-24",
      duration: 38 }
  - { work: "[[Préludes Book I - Debussy]]", tracks: "25-36",
      duration: 41 }
  - { work: "[[Piano Sonata No. 2 - Boulez]]", tracks: "37-40",
      duration: 31 }
performers:
  - { performer: "[[Maurizio Pollini]]", instrument: piano }
recording-type: studio
```

**Pattern — historic compilation.** Treat as a Recording-Release Edition that bundles multiple underlying Recordings:

```yaml
# Edition
kind: edition
type: recording-release
title: "The Complete Furtwängler Wartime Recordings"
of-bundle: 
  - "[[Beethoven 9 - Furtwängler 1942]]"
  - "[[Brahms 4 - Furtwängler 1943]]"
  # ...
label: "[[Music & Arts]]"
disc-count: 22
release-year: 2014
```

Use `of:` (singular) when the Edition is one Recording in different format; `of-bundle:` when it's many.

---

## 8. Improvisations, cadenzas, and jazz/classical hybrids

**Case.** Keith Jarrett's *Köln Concert*. Cecil Taylor solo improvisations. Continuo realization in Baroque performance. Improvised cadenzas (Mozart concertos performed with period-style improvised cadenzas vs. composer-written cadenzas).

**Decision.** For unscripted improvisation, the Recording *is* the work — don't force a Composition-Recording split. Use `kind: work, type: album` (the popular-music model) or introduce `type: improvisation`.

**Pattern.**

```yaml
kind: work
type: improvisation
title: "Köln Concert"
performer: "[[Keith Jarrett]]"
recorded: 1975-01-24
venue: "[[Opera House Köln]]"
parts: ["I", "II a", "II b", "II c"]
genres: [jazz-improvisation, solo-piano]
```

For cadenzas in concerto performance, the Recording note can carry a `cadenza-by:` field:

```yaml
# recording of a Mozart concerto
work: "[[Piano Concerto No. 20 in D minor K. 466 - Mozart]]"
cadenzas-used:
  - { movement: 1, by: "[[Ludwig van Beethoven]]" }
  - { movement: 3, by: "[[Ludwig van Beethoven]]" }
```

---

## 9. Period instruments, tuning, and pitch

**Case.** A Brüggen Beethoven cycle (period instruments) is interpretively distinct from a Karajan cycle (modern instruments). Bach on harpsichord vs. Bach on modern piano. Performances at A=415 vs. A=440. Mean-tone vs. equal temperament.

**Decision.** Track at Recording level via optional fields. These are interpretation metadata, not work metadata.

**Pattern.**

```yaml
# recording
instruments-period: period | modern | mixed | hip       # historically informed performance
tuning-pitch: "A=415"                                    # or A=440, A=430, etc.
temperament: equal | mean-tone | werckmeister-iii | kirnberger-iii | well-tempered | other
performance-practice: |
  Period instruments, A=430, ornamentation per Quantz.
```

Default-empty fields — most listeners won't fill these. The schema accommodates collectors who care.

---

## 10. Box sets, cycles, and integral recordings

**Case.** Karajan's three Beethoven symphony cycles (1962, 1977, 1982). Brendel's two Beethoven sonata cycles. The *Hyperion Schubert Edition* (40 discs of Lieder). A Bach cantatas pilgrimage (Gardiner's 56-disc *Bach Cantata Pilgrimage*).

**Decision.** These are Collections (curated/declared sets) made up of Recordings. The Collection note tracks the integral nature; individual Recording notes link back.

**Pattern.**

```yaml
# /Collections/Karajan Beethoven Cycle 1961-62.md
kind: collection
type: collection
collection-type: recording-cycle
title: "Karajan / BPO Beethoven Symphonies 1961-62"
curator: "[[Herbert von Karajan]]"
ensemble: "[[Berlin Philharmonic]]"
recorded-start: 1961
recorded-end: 1962
label: "[[Deutsche Grammophon]]"
release-year: 1963
items:
  - "[[Beethoven 1 - Karajan 1961]]"
  - "[[Beethoven 2 - Karajan 1961]]"
  # ... through 9
complete: true
covers-cycle: "[[Beethoven Symphonies]]"     # link to the Series of works being cycled
```

This lets you query "all complete Beethoven symphony cycles in the collection" and "the 1962 cycle's ratings vs. the 1977 cycle's."

---

## 11. Multi-composer works and pasticcios

**Case.** Handel's pasticcio operas (assembled from multiple composers' arias). Mozart's completion of his own *Requiem* by Süssmayr. Collaborative works like the *F-A-E Sonata* (Schumann/Brahms/Dietrich).

**Decision.** The Composition uses `composer:` for primary attribution and `co-composers:` for collaborators. For completions, use `completion-by:`. For pasticcios, document in `notes-composition:` since per-movement attribution gets noisy.

**Pattern.**

```yaml
title: "Requiem in D minor"
catalog: "K. 626"
composer: "[[Wolfgang Amadeus Mozart]]"
completion-by: "[[Franz Xaver Süssmayr]]"
completion-status: incomplete-at-death
completion-extent: |
  Requiem and Kyrie complete; Sequence and Offertory partially scored 
  (Mozart left vocal lines and bass with figures); Sanctus, Benedictus, 
  Agnus Dei composed by Süssmayr from sketches and instruction.
alternative-completions:
  - { by: "[[Robert Levin]]", year: 1991, name: "Levin completion" }
  - { by: "[[Richard Maunder]]", year: 1988, name: "Maunder completion" }
```

Recording notes specify which completion was used: `completion-used: "Süssmayr"` or `completion-used: "Levin 1991"`.

---

## 12. Anonymous, traditional, and chant repertoire

**Case.** Gregorian chant. Medieval polyphony with no known composer. Folk-tradition pieces. Hymns where attribution is uncertain.

**Decision.** Use `composer: "[[Anonymous]]"` (a real Person note that serves as a sentinel) or `composer: null` with `attribution-status: anonymous`. Record `tradition:` and `provenance:` instead.

**Pattern.**

```yaml
kind: work
type: composition
title: "Veni Creator Spiritus"
composer: null
attribution-status: anonymous
attributed-to-traditional: "[[Rabanus Maurus]]"      # legendary attribution
tradition: gregorian-chant
mode: 8                                              # for modal music
period: medieval
earliest-source: "9th century"
liturgical-function: "Pentecost; ordination"
```

For collected anthologies of chant or folk music, the Collection schema handles bundling.

---

## 13. Soundtracks and incidental music

**Case.** Prokofiev's *Alexander Nevsky* (film score later concert cantata). Mendelssohn's *Midsummer Night's Dream* (incidental music to Shakespeare). John Williams's *Star Wars* score. Music from theatrical productions reissued as concert suites.

**Decision.** When concert versions exist, treat them as derived Compositions (like arrangements of self). The film/play/show is `for-media:` linkage.

**Pattern.**

```yaml
title: "Alexander Nevsky (cantata)"
catalog: "Op. 78"
composer: "[[Sergei Prokofiev]]"
arrangement-of: "[[Alexander Nevsky (film score) - Prokofiev]]"
arrangement-type: concert-suite
for-media: "[[Alexander Nevsky (1938)]]"      # link to the film
```

The film score itself can be a Composition with `for-media:` set; if you don't care to model the score separately, put `for-media:` directly on the cantata note and skip the intermediate.

---

## 14. Solo and chamber recordings — escaping the conductor model

**Case.** A Glenn Gould Bach recording. The Emerson Quartet's late Beethoven cycle. A Lieder recital with pianist (Schubert's *Winterreise* — Fischer-Dieskau / Moore). A duo violin/piano recital.

**Decision.** Recording schema's `performers:` array (with `instrument:` and optional `role:`) replaces conductor/ensemble. Already in the base schema; flagged here as the canonical pattern.

**Pattern — Lieder.**

```yaml
work: "[[Winterreise D. 911 - Schubert]]"
conductor: null
ensemble: null
performers:
  - { performer: "[[Dietrich Fischer-Dieskau]]", instrument: voice, 
      voice-type: baritone, role: "singer" }
  - { performer: "[[Gerald Moore]]", instrument: piano, 
      role: "accompanist" }
```

**Pattern — string quartet.**

```yaml
work: "[[String Quartet No. 14 in C-sharp minor Op. 131 - Beethoven]]"
ensemble: "[[Emerson String Quartet]]"
performers: []                                     # ensemble carries the personnel
# Optional override if personnel differs from current roster:
performers-override:
  - { performer: "[[Eugene Drucker]]", instrument: violin, role: "first violin" }
  - { performer: "[[Philip Setzer]]", instrument: violin, role: "second violin" }
  - { performer: "[[Lawrence Dutton]]", instrument: viola }
  - { performer: "[[David Finckel]]", instrument: cello }
```

Use `performers-override:` only when the recording's personnel differs from the ensemble's current/canonical roster.

---

## 15. Liturgical, sacred, and functional music

**Case.** Bach cantatas tied to specific Sundays in the liturgical year. Catholic Mass settings. Anthems, motets, and hymns in specific liturgical contexts. Music written for specific occasions (coronations, funerals).

**Decision.** Add optional fields: `liturgical-function`, `liturgical-date`, `occasion`. Light-touch, fill when relevant.

**Pattern.**

```yaml
title: "Wachet auf, ruft uns die Stimme"
catalog: "BWV 140"
composer: "[[Johann Sebastian Bach]]"
form: cantata
sub-form: chorale-cantata
liturgical-function: "27th Sunday after Trinity"
liturgical-date: "1731-11-25"            # first performance
occasion: null
text-source: "[[Philipp Nicolai]] chorale (1599)"
```

For occasional pieces:

```yaml
title: "Music for the Royal Fireworks"
catalog: "HWV 351"
composer: "[[George Frideric Handel]]"
occasion: "Celebration of the Treaty of Aix-la-Chapelle"
occasion-date: 1749-04-27
commissioned-by: "[[King George II]]"
```

---

## 16. Premiere recordings, world premieres, and historical firsts

**Case.** "First commercial recording of Mahler 8" (Horenstein 1959). "World premiere of Boulez's *Répons*" (1981). "First recording with the original ending."

**Decision.** Recording note carries `premiere-recording:` flags. Composition note carries `recording-history:` for noteworthy firsts.

**Pattern.**

```yaml
# recording
premiere-recording: true
premiere-recording-type: world | commercial | studio | complete | uncut | original-instruments
notes-historical: |
  First commercial recording of the symphony. 
  Recorded after a public performance at the Royal Albert Hall.

# composition (optional)
recording-history:
  - { type: "world-premiere-recording", recording: "[[Mahler 8 - Horenstein 1959]]" }
  - { type: "first-period-instruments", recording: "[[Mahler 8 - Norrington 2010]]" }
```

---

## 17. Composer-as-performer

**Case.** Rachmaninoff's recordings of his own works. Bartók's piano rolls. Stravinsky conducting Stravinsky. Britten conducting Britten. Boulez conducting Boulez.

**Decision.** Just data — the same person fills `composer:` on the work and `conductor:` (or `performers:`) on the recording. Worth flagging as a queryable pattern: `recorded-by-composer: true` is a useful denormalized field.

**Pattern.**

```yaml
# recording
work: "[[Le Sacre du printemps - Stravinsky]]"
conductor: "[[Igor Stravinsky]]"
recorded-by-composer: true
notes-historical: "Stravinsky's 1960 stereo remake of his own work."
```

---

## 18. Genre and period boundaries

**Case.** Where does "early music" end and "Baroque" begin? Is Schoenberg "late Romantic" or "modern"? Crossover repertoire (Yo-Yo Ma's *Silk Road*, Hilary Hahn's *In 27 Pieces*). Minimalism, post-minimalism, spectralism.

**Decision.** Treat `period:` and `genres:` as overlapping multi-value fields, not exclusive categories. Don't try to taxonomize comprehensively up front.

**Suggested period values for classical specifically:**
`medieval | renaissance | baroque | classical | early-romantic | romantic | late-romantic | impressionist | modern | post-war | contemporary | minimalist | post-minimalist | spectralist | post-tonal`

A single work can carry multiple: `period: [late-romantic, modern]` is fine for transitional figures.

---

## 19. Live broadcast, radio, and bootleg recordings

**Case.** Off-air recordings of broadcast concerts. Soviet Melodiya tapes of Russian premieres. Bootleg Furtwängler. Cluytens' live cycles.

**Decision.** Already covered by `recording-type: broadcast | live | radio` in the base schema. Add `source-type:` for provenance when it matters to authenticity.

**Pattern.**

```yaml
recording-type: broadcast
source-type: off-air | direct-line | tape | wire | acetate | shellac | unknown
broadcast-station: "[[NDR]]"
broadcast-date: 1953-04-15
notes-source: |
  Off-air recording from NDR broadcast; some surface noise. 
  Music & Arts release sourced from collector's reel-to-reel copy.
```

---

## 20. Cross-references the schema must support

These aren't edge cases per se but are queries the cataloging schema must enable. Listed here so handoff documentation captures the *why*:

1. All recordings of a given Work, sortable by year/conductor/rating.
2. All works composed in a given period or by a given composer.
3. All recordings featuring a given performer (across ensembles, as soloist, as conductor).
4. All recordings made at a given venue (Jesus-Christus-Kirche, Wigmore Hall).
5. All works in a given catalog system, sortable numerically.
6. All complete cycles vs. incomplete sets.
7. All arrangements of a given source work.
8. All recordings using a specific edition (Nowak Bruckner 8, Süssmayr Mozart Requiem).
9. Period-instrument vs. modern-instrument recordings of the same work.
10. Recordings where the composer is also the performer.

These are analysis-layer concerns, deferred — but the schema's job at the cataloging stage is to make all of them *possible* without restructuring later.

---

## What the schema deliberately doesn't do

- **Per-track timing within multi-work recordings.** The `tracks:` field is free text. Time-coded track lists belong in MusicBrainz, not your vault.
- **Score location and edition tracking for personal scores.** A separate concern (could be modeled as `kind: edition, type: score-edition` if needed later).
- **Concert programs and recital attendance.** Different entity (a "concert" — the live event you went to). Add later if you catalog concert-going.
- **Performance practice scholarship.** Notes-fields can hold prose; structured modeling of HIP arguments isn't in scope.
- **Royalty and rights metadata.** Not a personal-use concern.
