# oddsdrop-feed

Public coupon feed consumed by the **OddsDrop** Android app (paper-trading *simulation*; no real betting, not advice).
The only data file is [`feed/coupons.json`](feed/coupons.json); the schema is [`feed.schema.json`](feed.schema.json).

Raw URL used by the app (default):

```
https://raw.githubusercontent.com/unflixtrt-sketch/oddsdrop-feed/main/feed/coupons.json
```

## Format (`feedVersion` 1)

```jsonc
{
  "feedVersion": 1,                 // integer; apps ignore feeds with a higher version than they know
  "generatedAt": 1792159283668,     // epoch ms (UTC) when this file was produced
  "coupons": [                      // oldest first, ~last 50 kept
    {
      "id": "a-20261016-1401-3d4598ab",   // stable unique string; an app imports each id at most once
      "createdAt": 1792159283668,          // epoch ms
      "legs": [                            // >= 3 legs, each from a DIFFERENT match
        {
          "matchId": 1064822,              // site match id (detay26.asp?id=...)
          "match": "Karbalaa - Al-Garaf",  // "Home - Away"
          "home": "Karbalaa", "away": "Al-Garaf",   // optional (else split from `match`)
          "kickoff": 1792162800000,        // epoch ms
          "market": "Çifte Şans",          // site market name (must be settleable by the app)
          "marketId": 2,                   // degisen26.asp turId/marketId
          "selection": "1 ve 0",           // site outcome name
          "odds": 1.22,                    // decimal odds at generation time (used by the app)
          "openOdds": 1.34, "dropPct": 9   // optional
        }
      ],
      "rationale": "…",                    // optional, <= 300 chars
      "score": 2.14                        // optional
    }
  ],
  "strategyWeights": {                 // optional, reserved for later; apps currently only store it
    "markets": { "Çifte Şans": 1.25 },
    "oddsRanges": [ { "min": 1.0, "max": 1.5, "weight": 1.0 } ]
  }
}
```

### App import rules (OddsDrop v1.2+)

* A coupon is imported when its `id` is not already in the app's local database, it has **>= 3 legs** (distinct matches),
  **all legs kick off in the future**, and the wallet balance covers the stake (default 1,000 TL, deducted at import).
  The coupon is opened with the feed's `odds` and labelled **Asistan** (locally generated ones are **Otomatik**).
* Invalid coupons/legs are skipped; an unreachable or invalid feed is ignored. The app sends `If-None-Match` (ETag).
* Markets/selections must use the site's own names so the app can settle them from the live-score endpoint.

## Publishing

`publish_feed.py` (kept outside this repo, in the maintainer's workspace) validates against `feed.schema.json`, merges new
coupons by `id`, keeps the last 50, and commits + pushes `feed/coupons.json`.
