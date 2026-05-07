# Schema Core

The complete entity model and frontmatter for the cataloging-phase vault. This document is the source of truth for what fields exist on what kind of note. Edge cases for classical music are in `02-classical-edge-cases.md` and reference fields defined here.

---

## Design principles (restated)

1. **Entities, not folders, are the structure.** Folders are organizational scaffolding for the file browser; the real graph lives in frontmatter links. Resist deep nesting.
2. **Every entity has stable identity before rich metadata.** A note with `kind` + `type` + `title` + `created` is enough to start linking. Onboarding fails when each new item demands 30 fields before save.
3. **Polymorphism via `type`, shared base via `kind`.** `kind` is the broad category (work, recording, edition, episode, collection, series, person, org, place, subject); `type` is the specialization within it.
4. **Containers are first-class.** Series, collections, box sets, anthologies, franchises are entities with their own notes.
5. **Acquisition is separate from the work.** A copy you own is a distinct entity from the work itself.

---

## Folder structure

```
/Media
  /Works
    /Compositions             musical works
    /Books
    /Films
    /TV                       series-level
    /Games
    /Albums                   popular-music albums (work-level)
    /Articles
    /Podcasts                 show-level
    /Comics
    /Performances             plays, operas, ballets (the work)
    /Other
  /Recordings                 musical performances of compositions
  /Editions                   specific editions/releases/manifestations
  /Episodes                   TV/podcast individual episodes (lazy-create)
  /Productions                opera/ballet stagings (optional, see edge cases)
  /Collections                box sets, curated lists, recording cycles
  /Series                     intrinsic multi-part containers
  /People
    /Composers
    /Authors
    /Directors
    /Performers
    /Conductors
    /Other-Creators
  /Organizations
    /Ensembles
    /Studios                  film/game studios
    /Publishers
    /Labels
    /Networks
  /Places
    /Venues
    /Studios-Recording
  /Subjects                   topics/themes/movements warranting notes
/Templates
/Attachments
/_Inbox                       capture-first staging
```

`_Inbox` exists so classification never blocks capture. Triage in batches.

---

## Universal base frontmatter

Every note has these fields. Most are optional at create time.

```yaml
---
kind: work | recording | edition | episode | production | collection | series | person | org | place | subject
type: <specialization>
title: ""
sort-title: ""              # title with leading articles stripped
aliases: []                 # alternate titles, translations, abbreviations

ids:
  wikidata: null
  musicbrainz: null
  imdb: null
  tmdb: null
  isbn: null
  oclc: null
  goodreads: null
  igdb: null
  asin: null
  doi: null

created: 2026-05-06
source: ""
confidence: confirmed | tentative | stub
needs-review: false

related: []
subjects: []
tags: []

status: queued | active | finished | abandoned | reference | owned-unread
acquired: null
rating: null
notes: ""
---
```

**Minimum viable note:** `kind`, `type`, `title`, `created`. Everything else fillable later.

`confidence` and `needs-review` are doing real work — they let aggressive capture stay honest about incompleteness, and they make "what needs filling in" a queryable set later.

---

## Work specializations

### Composition (musical work)

```yaml
---
kind: work
type: composition
title: ""
sort-title: ""
catalog: ""                 # display form: "Op. 125", "BWV 1080", "K. 466", "Hob. I:101"
catalog-system: ""          # enum value; see 07-catalog-systems.md for full list
catalog-number: null        # numeric portion only, for sorting
catalog-suffix: ""          # variant letter ("a", "b") or sub-version (".1") if applicable
catalog-group: ""           # for group-and-number systems (Hoboken: "I", "XVI")
catalog-section: ""         # for systems with appendices ("Anh. I", "Anh. II", "Anh. III")
catalog-edition: null       # which edition: "BWV3", "K6", "Sz" — see 07-catalog-systems.md
catalog-alternates: []      # array of {system, number, edition?} for alternate systems
nickname: ""                # "Choral", "Eroica", "Jupiter"
composer: ""                # link
co-composers: []
arranger: null              # for arrangements; see edge case 4
arrangement-of: null        # link to source work
arrangement-type: null      # orchestration | transcription | reduction | etc.
arrangement-year: null
librettist: null
lyricist: null
text-source: ""
based-on: null

# Attribution (see edge case 3)
attribution-status: confirmed | doubtful | spurious | anonymous | fragmentary | lost | reconstructed
attributed-to: null
attributed-also-to: []
attributed-to-traditional: null
completion-by: null
completion-status: null     # complete | incomplete-at-death | abandoned | sketch-only | fragmentary
completion-extent: ""
alternative-completions: []
sources-status: ""

# Temporal
composed-start: null
composed-end: null
premiered: null
premiere-venue: null
premiere-conductor: null

# Structural
form: ""                    # symphony, sonata, concerto, lied, opera, mass, etc.
sub-form: null
key: ""
mode: null                  # for modal music: 1-8
instrumentation: []
duration-typical: null
movements: []               # array of objects: {n, title, tempo, key}
parts-count: null           # for cycles
parts-summary: ""
has-named-parts: false
parts-promoted: []          # parts broken out into their own notes

# Versions (see edge case 5)
versions: []
default-version: null

# Liturgical / occasional (see edge case 15)
liturgical-function: null
liturgical-date: null
occasion: null
occasion-date: null
commissioned-by: null

# Tradition (see edge case 12)
tradition: null
earliest-source: null

# For media (see edge case 13)
for-media: null

# Period & genre
period: []                  # array; can carry multiple
genres: []

# Containers
part-of: null               # link to series
part-of-position: null
arrangements: []            # back-references to derivative works
recording-history: []       # noteworthy firsts
---
```

### Book

```yaml
---
kind: work
type: book
title: ""
sort-title: ""
original-title: ""
authors: []
co-authors: []
translators: []             # canonical translator if at work-level
illustrators: []
editors: []
introduction-by: []

original-language: ""
written-start: null
written-end: null
first-published: null
fictional: true | false

form: novel | novella | short-story | poetry-collection | essay-collection | play | screenplay | nonfiction | memoir | biography | reference | anthology
genres: []
themes: []
setting-time: ""
setting-place: ""

part-of: null
part-of-position: null
---
```

### Film

```yaml
---
kind: work
type: film
title: ""
sort-title: ""
original-title: ""
directors: []
writers: []
based-on: null
producers: []
cinematographers: []
editors: []
composers: []
production-designers: []
principal-cast: []          # array of {performer, role}

studios: []
distributors: []
release-year: null
release-date: null
country: ""
original-language: ""
runtime: null               # minutes
aspect-ratio: ""
color: color | b&w | mixed
sound: mono | stereo | surround | silent

genres: []
themes: []

part-of: null
part-of-position: null
---
```

### TV series

```yaml
---
kind: work
type: tv-series
title: ""
sort-title: ""
creators: []
showrunners: []
principal-cast: []
networks: []
studios: []

first-aired: null
last-aired: null
status-broadcast: ended | ongoing | cancelled | hiatus | upcoming
seasons-count: null
episodes-count: null
episode-runtime: null
country: ""

genres: []
themes: []

seasons: []                 # array of {n, year, episodes, title}
---
```

### Game

```yaml
---
kind: work
type: game
title: ""
sort-title: ""
developers: []
publishers: []
directors: []
writers: []
designers: []
composers: []
artists: []

released: null
platforms-original: []
platforms-available: []
engine: ""
genres: []
themes: []
modes: []                   # single-player, multiplayer, co-op, mmo
perspectives: []            # first-person, third-person, isometric, top-down, etc.
playtime-typical: null

part-of: null
part-of-position: null
---
```

### Album (popular-music model)

For pop / rock / jazz / electronic — where the album is the authored unit.

```yaml
---
kind: work
type: album
title: ""
sort-title: ""
artists: []
producers: []
released: null
label: null
recorded-start: null
recorded-end: null
recording-venues: []
genre: []
length: null                # minutes
tracks: []                  # array of {n, title, length}
personnel: []               # for jazz; array of {performer, instrument}
---
```

### Article

```yaml
---
kind: work
type: article
title: ""
authors: []
published-in: ""
published-date: null
url: ""
doi: null
length: null                # words
form: essay | review | interview | feature | academic-paper | newsletter
genres: []
themes: []
---
```

### Podcast (show-level)

```yaml
---
kind: work
type: podcast
title: ""
hosts: []
network: null
first-aired: null
status-broadcast: ongoing | ended
episode-runtime: null
genres: []
---
```

### Comic / graphic novel

```yaml
---
kind: work
type: comic
title: ""
writers: []
artists: []
publishers: []
form: ongoing-series | limited-series | graphic-novel | one-shot | strip
first-published: null
last-published: null
issues-count: null
volumes-count: null
genres: []
---
```

### Performance work (play, opera, ballet, musical)

```yaml
---
kind: work
type: performance-work
title: ""
sub-type: play | opera | ballet | musical
playwright: null
composer: null
librettist: null
choreographer: null
written: null
premiered: null
language: ""
acts: null
duration-typical: null
genres: []
---
```

### Improvisation (see edge case 8)

```yaml
---
kind: work
type: improvisation
title: ""
performer: ""               # link
recorded: null
venue: null
parts: []
genres: []
---
```

---

## Recording

```yaml
---
kind: recording
type: recording
title: ""
work: ""                    # link; null when multi-work
works: []                   # array of links or {work, tracks, duration}
version: null               # see edge case 5: composition version id
score-edition: null         # Nowak, Haas, Urtext, etc.

# Performance forces
conductor: null
ensemble: null
soloists: []                # array of {performer, role, voice-type?}
choir: null
performers: []              # for chamber/solo
performers-override: []     # when ensemble's roster differs
instruments: []
director: null              # for staged works
production: null            # link to production note
cadenzas-used: []           # array of {movement, by}
completion-used: null       # for incomplete-work recordings

# Recording specifics
recorded-start: null
recorded-end: null
recording-venue: null
producer: null
engineer: null
recording-type: studio | live | broadcast | rehearsal | radio
recording-medium: audio | video
mono-stereo: mono | stereo | surround
source-type: null           # off-air | direct-line | tape | etc.
broadcast-station: null
broadcast-date: null
notes-source: ""

# Performance practice (see edge case 9)
instruments-period: period | modern | mixed | hip
tuning-pitch: null          # "A=415", "A=440"
temperament: null           # equal | mean-tone | werckmeister-iii | etc.
performance-practice: ""

# Historical
premiere-recording: false
premiere-recording-type: null
notes-historical: ""
recorded-by-composer: false

# Containers
part-of-collection: null    # link to collection
---
```

---

## Edition

The acquisition / manifestation entity. One Edition per physical-or-digital copy you own or care to track.

```yaml
---
kind: edition
type: book-edition | film-edition | game-edition | recording-release | comic-edition | score-edition
of: ""                      # link to work or recording (singular)
of-bundle: []               # for compilations bundling multiple recordings/works

# Bibliographic / release
publisher: null             # or label, or studio
imprint: null
edition-name: ""
edition-number: null
translator: null
introduction-by: null
illustrator: null
cover-artist: null
release-year: null
release-date: null
catalog-number: ""
isbn: null
asin: null
barcode: null

# Format — book
format: paperback | hardcover | ebook | audiobook | leatherbound | trade-paperback | mass-market | omnibus | facsimile
language: ""
pages: null
size: ""
weight: null
audio-narrator: null
audio-runtime: null

# Format — film
medium-film: theatrical | bluray | uhd | dvd | streaming | vhs | laserdisc | digital-purchase
cut: theatrical | directors | extended | criterion | uncut
aspect-ratio: null
audio-tracks: []
subtitles: []
region: null
distributor: null
spine-number: null

# Format — game
medium-game: physical | digital
platform: null
storefront: steam | gog | epic | psn | xbox-store | nintendo-eshop | physical | other
edition-type: standard | deluxe | collectors | goty
drm: none | steam | other
region-game: null

# Format — recording release
medium-music: cd | vinyl | sacd | cassette | digital-download | hi-res-download | streaming
remaster-year: null
disc-count: null
remastering-engineer: null

# Acquisition
acquired: null
acquired-from: null
acquired-price: null
condition: new | very-good | good | fair | poor
location: ""                # "Shelf B3", "Plex", "Roon", "Steam"
provenance: ""              # gift, inheritance, find
signed: false
notes-edition: ""
---
```

Templater splits this into one template per edition type so only relevant fields appear at create time.

---

## Episode (lazy-create)

```yaml
---
kind: episode
type: tv-episode | podcast-episode
title: ""
series: ""                  # link
season: null
episode-number: null
absolute-number: null
aired: null
runtime: null
directed-by: null
written-by: null
guest-cast: []
---
```

---

## Production (optional, for staged works)

See edge case 6.

```yaml
---
kind: production
type: production
of: ""                      # link to performance-work
director: null
set-designer: null
costume-designer: null
choreographer: null
conductor: null             # if a specific musical leadership is part of the production
premiere-date: null
venue: null
run-end: null
notes-production: ""
---
```

---

## Collection

User-curated or publisher-curated groupings that aren't intrinsic series.

```yaml
---
kind: collection
type: collection
title: ""
collection-type: box-set | publisher-set | personal-curation | award-list | reading-list | listening-project | exhibition | recording-cycle
curator: null
released: null
ensemble: null              # for recording cycles
recorded-start: null
recorded-end: null
label: null
release-year: null
items: []                   # explicit array of links
complete: false
covers-cycle: null          # link to series of works being cycled, if applicable
notes-collection: ""
---
```

Both directions of membership are stored: `items:` on Collection, `part-of-collection:` on child notes. Redundant on purpose — neither query path becomes fragile.

---

## Series

Intrinsic multi-part containers (trilogies, numbered series, franchises, symphony cycles).

```yaml
---
kind: series
type: series
title: ""
series-type: book-series | film-series | tv-series-arc | game-series | franchise | symphony-cycle | concerto-set | opera-cycle
creators: []
parts: []                   # array of {n, title, year, link}
canonical-order: chronological | publication | creator-preferred
parent-franchise: null
---
```

---

## People

### Composer

```yaml
---
kind: person
type: composer
name: ""
sort-name: ""
born: null
born-place: ""
died: null
died-place: ""
nationality: ""
period: []
teachers: []
students: []
notable-works: []
catalog-systems-canonical: []   # see 07-catalog-systems.md for valid enum values
catalog-systems-alternate: []
catalog-edition-canonical: null # for systems with multiple editions: "BWV3", "K6"
also-known-as: []           # other roles: pianist, conductor, etc.
---
```

### Author

```yaml
---
kind: person
type: author
name: ""
sort-name: ""
born: null
died: null
nationality: ""
languages-written: []
movements: []
notable-works: []
also-known-as: []
---
```

### Director

```yaml
---
kind: person
type: director
name: ""
born: null
died: null
nationality: ""
notable-works: []
collaborators: []
also-known-as: []
---
```

### Performer

Covers actors, instrumentalists, singers, narrators.

```yaml
---
kind: person
type: performer
name: ""
sort-name: ""
born: null
died: null
nationality: ""
roles: []                   # pianist, violinist, actor, singer, narrator, etc.
voice-type: null            # soprano | mezzo | tenor | bass | etc.
instruments: []
specialties: []
era: []
primary-ensembles: []
---
```

### Conductor

```yaml
---
kind: person
type: conductor
name: ""
born: null
died: null
nationality: ""
primary-ensembles: []
specialties: []
also-known-as: []
---
```

### Other-creators (catch-all)

```yaml
---
kind: person
type: creator
name: ""
roles: []
born: null
died: null
nationality: ""
notable-works: []
---
```

---

## Organizations

### Ensemble

```yaml
---
kind: org
type: ensemble
name: ""
ensemble-type: orchestra | quartet | trio | choir | chamber-orchestra | early-music-ensemble | jazz-band | rock-band
formed: null
disbanded: null
home-venue: null
home-city: ""
chief-conductors: []        # array of {person, from, to}
size: null
notable-members: []
---
```

### Studio (film/game)

```yaml
---
kind: org
type: studio
name: ""
studio-type: film | game | recording | design
founded: null
founders: []
country: ""
notable-works: []
---
```

### Publisher / Label / Network

```yaml
---
kind: org
type: publisher | label | network
name: ""
parent: null
founded: null
country: ""
specialties: []
imprints: []
---
```

---

## Places

### Venue

```yaml
---
kind: place
type: venue
name: ""
venue-type: concert-hall | opera-house | theater | recording-studio | cinema | arena | club
city: ""
country: ""
opened: null
architect: null
capacity: null
home-to: []
---
```

### Recording studio

```yaml
---
kind: place
type: recording-studio
name: ""
city: ""
country: ""
opened: null
notable-recordings: []
---
```

---

## Subject

```yaml
---
kind: subject
type: subject
name: ""
subject-type: genre | movement | theme | period | technique | discipline
parent: null
related: []
description: ""
---
```

---

## Sentinel notes

A few pre-created notes serve as link targets for special cases:

- `[[Anonymous]]` — kind: person, type: composer, sentinel for anonymous works
- `[[Traditional]]` — sentinel for folk/traditional pieces with no attribution
- `[[Various Artists]]` — for compilation editions that don't bundle by performer

These let `composer:`, `artists:`, etc. always be valid links rather than null/empty special cases.
