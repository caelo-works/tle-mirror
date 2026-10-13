# tle-mirror

A periodic snapshot of public orbital data, so tools on networks that cannot
reach `celestrak.org` can still fetch it. A scheduled GitHub Action downloads
the data, validates it, commits it to `tle/` and publishes a SHA-256 manifest
alongside it. Files are served over HTTPS by `raw.githubusercontent.com`:

```
https://raw.githubusercontent.com/caelo-works/tle-mirror/main/tle/<file>
```

## What is mirrored

Every source is fetched over HTTPS.

| File(s) | Source | Content |
| --- | --- | --- |
| `tle/<group>.tle` | CelesTrak GP groups (`https://celestrak.org/NORAD/elements/gp.php?GROUP=<group>&FORMAT=tle`) | Curated element sets: `active`, `stations`, `starlink`, `oneweb`, `last-30-days`, `visual`, `galileo`, `gps-ops`, `glo-ops`, `science`, `geo`, `iridium-NEXT`, `planet`, `cosmos-2251-debris`, `iridium-33-debris`, `fengyun-1c-debris`, `cosmos-1408-debris` |
| `tle/catalog.tle` | CelesTrak GP queries per launch year (`https://celestrak.org/NORAD/elements/gp.php?INTDES=<year>&FORMAT=tle`, 1957 to the current year) | The full public GP catalog, including rocket bodies and defunct payloads that the curated groups do not cover |
| `tle/classfd.tle` | McCants classified elements (`https://www.mmccants.org/tles/classfd.zip`) | Amateur-tracked objects absent from the public GP catalog |
| `tle/satcat.csv` | CelesTrak SATCAT (`https://celestrak.org/pub/satcat.csv`) | Satellite catalog: names, owners/countries, launch and decay data |
| `tle/updated.txt` | — | UTC timestamp of the last refresh |
| `tle/SHA256SUMS` | — | SHA-256 manifest of all the files above |

## Schedule

The workflow runs every 8 hours (at minute 17 of 00:00, 08:00 and 16:00 UTC)
and can also be started manually. A new commit is made only when something
changed.

## Validation before replacement

A freshly downloaded file only replaces the previous copy if it passes these
checks; otherwise the previous copy is kept as is.

- **GP groups**: the payload must contain element lines (lines starting with
  `1 ` and `2 `). This rejects HTML error pages served with HTTP 200. The run
  fails if no group at all could be fetched.
- **Full catalog**: every launch-year query must succeed (HTTP 200 with element
  lines, or HTTP 404 for a year with no remaining objects), and the assembled
  catalog must hold at least 20000 objects.
- **McCants classified elements**: the archive must unpack to a payload
  containing element lines.
- **SATCAT**: the first line must be the CSV header containing `NORAD_CAT_ID`.

## Integrity manifest

At every snapshot the workflow regenerates `tle/SHA256SUMS` over every file in
`tle/` (except the manifest itself and `.gitkeep`), checks it against the files
with `sha256sum -c`, and commits it in the same commit as the files it
describes. The format is the standard `sha256sum` output, one line per file,
sorted by name, filenames relative to `tle/`:

```
<64 lowercase hex digits>  <filename>
```

To verify downloaded files, put them in a directory together with the manifest
and run:

```sh
base=https://raw.githubusercontent.com/caelo-works/tle-mirror/main/tle
curl -fsSO "$base/SHA256SUMS"
curl -fsSO "$base/stations.tle"
curl -fsSO "$base/satcat.csv"
sha256sum -c --ignore-missing SHA256SUMS
```

Right after an update, `raw.githubusercontent.com` can briefly serve files from
different commits (for example a new manifest with an old data file). On a
mismatch, a consumer should refetch the manifest and the file once before
treating the file as invalid.

## Consumers

The [Sky Intruders](https://github.com/caelo-works/sky-intruders) script for
PixInsight uses this mirror as a fallback when `celestrak.org` does not answer,
and verifies every file it downloads from here against `SHA256SUMS`.

## Data credit

Orbital elements and the satellite catalog are produced by
[CelesTrak](https://celestrak.org) (Dr. T.S. Kelso). The classified element
sets in `classfd.tle` are compiled by Mike McCants. This repository only
re-serves their data: element sets are not modified (the per-year queries are
concatenated into `catalog.tle`, and `classfd.zip` is unpacked).
