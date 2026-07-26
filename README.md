# Collection Tracker

A single-page tracker for any collection. Load a catalog of everything that exists, load your own file of what you own, and tick things off. Items you don't have render faded and greyed out so gaps are obvious at a glance.

Everything runs in the browser. No server, no database, no accounts. Your collection lives in a JSON file you keep.

## No catalogs included

This repo ships the app only. You bring your own catalog and collection files — see the schema below.

## Using it

1. **Load catalog** — pick a catalog file
2. **Load my collection** — pick your collection file (or skip it; you'll start from zero)
3. Tap any tile to toggle owned
4. **Save my collection** — writes your file back out (`Ctrl`/`Cmd`+`S` also works)

On Chrome and Edge, saving overwrites the file in place. Other browsers download a fresh copy.

You can also drag both files anywhere onto the page. It works out which is which.

Other controls: a search box, filters for **Have** / **Missing** / **Variants**, and a **Group** row that appears automatically if your catalog uses categories. **Check images** tests every image URL and reports what loaded, grouped by domain — useful when tiles come up blank.

Nothing is shared between people. Each person keeps their own collection file; there's no syncing and nobody sees anyone else's.

---

## Catalog format

The file the app reads to know what exists. Minimum viable version:

```json
{
  "items": [
    { "id": "thing-1", "name": "First Thing", "image": "images/first.jpg" },
    { "id": "thing-2", "name": "Second Thing", "image": "https://example.com/second.jpg" }
  ]
}
```

A bare array also works: `[{ "id": "...", "name": "...", "image": "..." }]`

### Item fields

| Field | Required | What it does |
|---|---|---|
| `id` | recommended | Unique key. Your collection file references these. Falls back to `name`, then a positional id — but set it explicitly, or reordering the catalog will scramble what's ticked. |
| `name` | yes | Shown on the tile. |
| `image` | yes | URL or path relative to `index.html`. Blank shows a placeholder. |
| `category` | no | Groups items. Creates the Group filter row. |
| `line` | no | Shown in the tile's meta line. |
| `show` | no | Also shown in the meta line. `series` works as an alias. |
| `release` | no | Not displayed, but searchable. Any format. |
| `printing` | no | `original` (default) or anything else. Non-original counts as a variant and gets a badge. |
| `printingLabel` | no | Badge text, e.g. `Reissue`, `SDCC`, `Limited Color`. |

Three `printing` values get their own badge colour: `reissue`, `special`, and `set`. Anything else uses the default.

### Catalog metadata

Optional top-level fields that configure the page, so one `index.html` serves any collection:

```json
{
  "title": "My Collection\nOf Things",
  "eyebrow": "Small text above the title",
  "subtitle": "Sentence under the title.",
  "categoryLabels": {
    "cat-a": "Category A",
    "cat-b": "Category B"
  },
  "items": []
}
```

`\n` in `title` becomes a line break. `categoryLabels` maps each `category` value to a readable name for the filter chips — without it, chips show the raw value.

### Fuller example

```json
{
  "title": "Widget\nCollection",
  "eyebrow": "Acme Corp · Collectables",
  "subtitle": "Tap a tile to mark it owned; faded tiles are still missing.",
  "categoryLabels": { "standard": "Standard line", "deluxe": "Deluxe line" },
  "items": [
    {
      "id": "wid-001",
      "name": "Red Widget",
      "line": "Widget Series",
      "category": "standard",
      "show": "Series One",
      "release": "2021-04",
      "printing": "original",
      "image": "images/red-widget.jpg"
    },
    {
      "id": "wid-001-reissue",
      "name": "Red Widget",
      "line": "Widget Series",
      "category": "standard",
      "show": "Series One",
      "release": "2024-09",
      "printing": "reissue",
      "printingLabel": "Reissue",
      "image": "images/red-widget.jpg"
    }
  ]
}
```

Extra fields you add are preserved in the file and ignored by the app — handy for notes to yourself about where data came from or how confident you are in it.

---

## Collection format

What you own. Just a list of ids from the catalog:

```json
{
  "collection": "My shelf",
  "updated": "2026-07-26",
  "count": 2,
  "owned": ["thing-1", "thing-2"]
}
```

A bare array of ids works too: `["thing-1", "thing-2"]`

Only `owned` matters. `collection`, `updated`, and `count` are written when you save and are there for your benefit. Ids that don't match anything in the catalog are ignored rather than erroring, so a stale collection file won't break.

---

## Notes on images

Images can be remote URLs or files committed alongside `index.html`.

Remote URLs are quicker to set up but you don't control them: links rot when the host reorganises, and some servers refuse to serve images to other sites, which shows up as blank tiles. The app sends no referrer to improve the odds, and **Check images** will tell you which domains are failing.

Local images in an `images/` folder with relative paths are slower to assemble but won't break.

---

## Hosting

Works from a local folder — open `index.html` directly. Some browsers restrict local files more than others, so if images misbehave, try serving it over http instead.

For GitHub Pages: put `index.html` at the repo root, then **Settings → Pages → Deploy from a branch → main → / (root)**.

---

## Limitations

- No shared or synced state; everyone tracks separately
- No browser storage — if you don't save, changes are lost on refresh
- Catalog accuracy is entirely down to your data; the app just renders it
