# SB Craft library

Data libraries for the **SB Craft** SketchUp extension (Sivá Brada Craft): manufacturer catalogues (PDF),
hardware packs and material libraries. This repository contains **data only, never code**.

The extension never downloads anything by itself. The user opens *Settings → System → Downloadable libraries*,
clicks **Refresh list** (downloads `katalog.json`) and then **Download** for a single library. Every file is
verified by SHA-256 and size before anything is installed; archives may contain only data file types.

## How it is published

* One library = one **GitHub release** of this repository, tag `<kind>-<id>-<version>`
  (e.g. `materials-ral_classic-1.0`), the file is the release asset.
* The list `katalog.json` is published at `https://sbcraft.eu/kniznice/katalog.json`
  (fallback: `katalog.json` in the `main` branch of this repository).
* Libraries of third-party manufacturers are distributed only with the manufacturer's consent
  (`"consent": true`). Without consent the list contains only a link to the manufacturer's official download page;
  the user downloads the file there and adds it in SB Craft with **Add downloaded file…**.

## `katalog.json` (format `sb-craft-library-catalog`, version 1)

```json
{
 "format": "sb-craft-library-catalog",
 "version": 1,
 "updated": "2026-10-29",
 "items": [
  {
   "id": "ral_classic",                 // [a-z0-9][a-z0-9_-]{0,63}, stable – never renamed
   "kind": "materials",                 // catalog | hardware | materials
   "vendor": "SB Craft",
   "name": {"sk": "…", "en": "…"},      // sk en cs hu de fr pl it es pt (missing → English)
   "description": {"en": "…"},          // optional
   "version": "1.0",                    // compared numerically per dot-separated part
   "languages": [],                     // catalog: language versions contained in the file
   "size": 424473,                      // bytes, must match
   "sha256": "2b42…",                   // lowercase hex, must match
   "url": "https://github.com/SBCraft-EU/sbcraft-library/releases/download/materials-ral_classic-1.0/ral_classic-1.0.zip",
   "date": "2026-10-29",
   "consent": true,                     // false → not downloadable, only vendor_url
   "vendor_url": "https://…",           // optional, manufacturer's page
   "min_sb_craft": "1.50"               // optional
  }
 ]
}
```

Additional fields for `kind: catalog`: `catalog` (catalogue id from the extension's register, e.g. `hranipex_2025`)
and optionally `file` (original PDF file name). Allowed hosts for `url`: `sbcraft.eu`, `github.com`,
`objects.githubusercontent.com`, `release-assets.githubusercontent.com`, `raw.githubusercontent.com` (HTTPS only).

## Assets

| kind | asset | installed to (`%APPDATA%\SB Craft\…`) | allowed file types |
|---|---|---|---|
| `catalog` | `.pdf` (one language) or `.zip` of PDFs | `Catalogs\<manufacturer>\<original name>` | pdf |
| `hardware` | `.zip` with `manifest.json` (`sb-craft-hardware-pack`) | `Hardware\Packs\<id>` | json skp png jpg jpeg svg webp gif pdf txt |
| `materials` | `.zip` with `.skm` (+ `manifest.json`, `sb-craft-material-pack`) | `Materials\Packs\<id>` | skm json png jpg jpeg svg webp gif pdf txt |

ZIP rules: no `..`, absolute paths, drive letters, symbolic links, encryption or ZIP64; no duplicate names;
the manifest `id` must equal the item `id` and every file listed in the manifest must match its size and SHA-256.
A single top-level folder in the archive is allowed.

## Tools

`tools/make_catalog.py` (Python 3, standard library only):

```
python tools/make_catalog.py pack-materials <folder> <id> <version> build/<id>-<version>.zip --name-en "…" --name-sk "…"
python tools/make_catalog.py pack-hardware <pack folder> build/<id>-<version>.zip
python tools/make_catalog.py build sources.json katalog.json
gh release create <tag> build/<asset> --repo SBCraft-EU/sbcraft-library --title "<name> <version>"
```

ZIP files are deterministic (fixed timestamps, sorted entries), so the same input always gives the same SHA-256.

## Privacy

When the list is refreshed or a library is downloaded, the servers (sbcraft.eu, GitHub) see only the IP address
and the request. The extension sends nothing else.
