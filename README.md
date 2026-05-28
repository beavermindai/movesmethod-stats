# movesmethod-stats

Live operational metrics for the MovesMethod × BeaverMind Phase 1 coaching-evaluation system.

Fed live into the proposal page at https://proposal.beavermind.ai/movesmethod (Phase 1 proof section).

## Raw fetch URL

```
https://raw.githubusercontent.com/beavermindai/movesmethod-stats/main/phase1-stats.json
```

## Schema

| Key | Type | Meaning |
|---|---|---|
| `schema_version` | string | Pin to `"1.0"` |
| `updated_at` | ISO 8601 string | UTC timestamp of last write |
| `evaluations_total` | integer | Cumulative evaluations completed by Phase 1 |
| `active_clients_daily` | integer | Distinct clients scored in last 24h |
| `coaches_monitored` | integer | Distinct coaches in scope |
| `days_running` | integer | Days since Phase 1 went self-driving |
| `cost_per_eval_usd` | number | Avg cost per evaluation (USD) |
| `max_minutes_per_eval` | integer | Upper-bound end-to-end latency (min) |

## How to update

**Manual** — edit `phase1-stats.json`, commit, push. Vercel proposal page picks it up on next view (no cache).

**Automated (recommended)** — schedule a GitHub Action that queries the Phase 1 Supabase view + overwrites the file. Skeleton:

```yaml
# .github/workflows/refresh.yml
on:
  schedule: [{cron: "0 */6 * * *"}]
  workflow_dispatch:
jobs:
  refresh:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: |
          curl -s "$SUPABASE_URL/rest/v1/rpc/movesmethod_phase1_stats" \
            -H "apikey: $SUPABASE_KEY" \
            -H "Authorization: Bearer $SUPABASE_KEY" > phase1-stats.json
        env:
          SUPABASE_URL: ${{ secrets.SUPABASE_URL }}
          SUPABASE_KEY: ${{ secrets.SUPABASE_KEY }}
      - run: |
          git config user.name 'stats-bot'
          git config user.email 'stats-bot@beavermind.ai'
          git add phase1-stats.json
          git diff --cached --quiet || (git commit -m "auto: refresh stats" && git push)
```

Define a Postgres function `movesmethod_phase1_stats` returning the schema above.

## Consumer

Single fetch from the proposal page — `STATS_URL` constant in `proposals/movesmethod.html`. No cache headers required; Vercel proxies pass-through.
