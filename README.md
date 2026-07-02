# Star Wars Codex — Open Data

An open, machine-readable snapshot of the **Star Wars Codex** catalog: every
published book, comic, film, television episode, game, and audio release, with
in-universe dates, eras, creators, and edition identifiers.

This repository is the data, in the open. The Codex is a labor-of-love reference
project, and the catalog it's built on shouldn't be locked behind a single
website. So it lives here too — plain JSON, one file per work, regenerated from
the live catalog and free for anyone to read, fork, and build on.

> **Snapshot, not an API.** This is a static export refreshed on a schedule, not
> a live endpoint. Clone it, diff it, script against it. Each refresh is also
> tagged `data-YYYY-MM-DD`, so you can pin or compare specific days.

## What's inside

```
data/
  works/<format>/<slug>.json      one file per published work, grouped by format
  timelines/<slug>.json           curated in-universe timelines
  reading-orders/<slug>.json      curated reading orders
  viewing-orders/<slug>.json      curated viewing orders
  bundles/
    works.json                    every work in a single JSON array (sorted by slug)
    works.ndjson                  every work, one JSON object per line (stream-friendly)
schema/
  work.schema.json                the published contract for a work
  timeline.schema.json
  reading-order.schema.json
  viewing-order.schema.json
meta/
  counts.json                     record counts per collection
  generated-at.txt                ISO timestamp of the last refresh
```

Works are grouped into these format folders:

`animated-short` · `audio-drama` · `book` · `movie` · `periodical` ·
`periodical-issue` · `short-story` · `tv-episode` · `tv-season` · `tv-series` ·
`video-game`

## A work, in JSON

Each file is one work. The `slug` is the stable public key and matches the
filename.

```json
{
  "slug": "rogue-one-a-star-wars-story",
  "title": "Rogue One: A Star Wars Story",
  "displayTitle": "Rogue One",
  "format": "movie",
  "canonStatus": "canon",
  "era": "reignOfTheEmpire",
  "releaseDate": "2016-12-15T00:00:00.000Z",
  "releaseStatus": "released",
  "releaseDisplayState": "released",
  "inUniverseDate": { "startYear": -1 },
  "movie": {
    "movieType": "theatrical",
    "subtitle": "A Star Wars Story",
    "runtimeMinutes": 133,
    "mpaaRating": "pg13"
  },
  "creators": [
    { "name": "Gareth Edwards", "role": "director", "slug": "gareth-edwards" },
    { "name": "Tony Gilroy", "role": "writer", "slug": "tony-gilroy" }
  ]
}
```

The full field list, types, and which fields are required live in
[`schema/work.schema.json`](schema/work.schema.json) (JSON Schema). Format-specific
detail hangs off a nested object keyed by family — `movie`, `periodical`,
`editions`, and so on.

Related works are linked by slug (`parentWorkSlug`, `relatedWorkSlugs`) so you
can walk the graph without any internal IDs.

## What's intentionally left out

The export is deliberately a **facts-only** view. To keep it clean, portable, and
privacy-respecting, it drops:

- internal database IDs, draft/review, and editorial workflow state
- images and media binaries
- outbound/store links and any third-party service identifiers
- anything about *where* a fact came from

People aren't published as separate records; a work's `creators` are rolled up
inline to `{ name, role, slug }`. Edition identifiers that are genuinely part of
the public record — ISBN, ASIN, UPC — **are** included, under `editions` /
`editionIds`.

## Using it

Grab the whole catalog as one array:

```bash
curl -LO https://raw.githubusercontent.com/shawnphoffman/star-wars-codex-data/main/data/bundles/works.json
```

Stream it line-by-line with `jq` (using the NDJSON bundle):

```bash
# Every canon film, newest first
jq -r 'select(.format=="movie" and .canonStatus=="canon") | .releaseDate + "  " + .title' \
  data/bundles/works.ndjson | sort -r

# Count works by format
jq -r '.format' data/bundles/works.ndjson | sort | uniq -c | sort -rn
```

Or just read individual files — they're small, human-readable, and diff cleanly
between snapshots.

## How it's generated

An automated job reads the **published** catalog from the Star Wars Codex,
sanitizes each record against the schema above, and commits the result here on a
regular cadence. Every record is validated before anything is published: a
failed validation, a duplicate slug, or a stray credential-shaped string aborts
the whole run, so a bad snapshot never lands. No-op runs produce no commit.

## License

🚩 **Licensing is still being finalized.** The intent is for this catalog data to
be openly usable; a formal license will be added here before you rely on it for
anything redistributive. Until then, treat it as "openly shared, license
pending" and check back. Star Wars, character names, and related marks are
trademarks of Lucasfilm Ltd.; this is an unofficial fan reference and is not
affiliated with or endorsed by Lucasfilm or The Walt Disney Company.

## About

Built as part of the **Star Wars Codex** — an unofficial, fan-made reference
catalog of the Star Wars universe. <!-- 🚩 add public site URL here -->
