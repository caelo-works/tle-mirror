# tle-mirror

A periodic snapshot of [CelesTrak](https://celestrak.org) GP element sets (TLE),
so tools on networks that cannot reach `celestrak.org` can still fetch orbital
data. A scheduled GitHub Action fetches the element sets every 8 hours (GitHub's
runners reach CelesTrak fine) and commits them here; they are served over
`raw.githubusercontent.com`.

Used as a fallback source by [Sky Intruders](https://github.com/caelo-works/sky-intruders).

## Fetch a group

```
https://raw.githubusercontent.com/caelo-works/tle-mirror/main/tle/<group>.tle
```

e.g. [`tle/active.tle`](tle/active.tle), `tle/starlink.tle`, `tle/stations.tle`.
`tle/updated.txt` holds the last refresh timestamp (UTC).

Mirrored groups: `active`, `stations`, `starlink`, `oneweb`, `last-30-days`,
`visual`, `galileo`, `gps-ops`, `glo-ops`, `science`, `geo`, `iridium-NEXT`,
`planet`.
`tle/catalog.tle` is the full public GP catalog (all ~34k tracked objects, rocket bodies and defunct payloads included).

Data is © CelesTrak / Dr. T.S. Kelso; this repository only re-serves it.
