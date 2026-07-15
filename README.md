# pmarkets

Automated trading strategies for prediction markets.

## Edge Scanner v1

Pulls every open market from Kalshi and Polymarket (both public, no API
keys), filters for liquidity and time-to-resolution, then flags three
signal types:

- **FAVORITE** -- longshot-bias buys at 90-96c
- **ARB** -- cross-platform price gaps >4c after Kalshi fees
- **MOVER** -- 8+ point moves since your last scan, flagged for review,
  not auto-traded

Every tradeable signal (FAVORITE, ARB) comes pre-sized: worst-case loss
capped at 2% of bankroll, then quarter-Kelly on top of that. Paper
positions and P&L live in SQLite, broken out by signal type, so after 30
closed trades you'll know which signals actually pay.

### Setup

```
pip install -r requirements.txt
```

### Usage

```
python scanner.py scan
python scanner.py take 3 "your thesis"
python scanner.py resolve 1 won
python scanner.py pnl
```

`scan --json` emits machine-readable signals for downstream pipelines.
`scan --bankroll 5000` updates the stored bankroll used for sizing.

### Things to watch on the first live run

- This sandbox can't reach kalshi.com or polymarket.com, so the test
  suite mocks both APIs -- the first live run happens on your machine.
- Polymarket's Gamma prices lag the live CLOB book slightly.
- ARB matching is fuzzy title similarity -- it will flag pairs with
  different resolution rules. Verify before counting any arb as real.
- Cron the scan 2-3x daily for MOVER detection to work (it compares
  against the previous scan's snapshot).

### v2 candidates (once paper data comes in)

- Catalyst calendar
- CLOB-level prices instead of Gamma snapshots
- Auto-resolution checking

### Layout

```
scanner.py                 CLI entrypoint
edge_scanner/
  config.py                thresholds, fee rate, sizing constants
  kalshi_client.py          Kalshi public markets fetch
  polymarket_client.py      Polymarket Gamma public markets fetch
  matching.py                fuzzy cross-platform title matching (ARB)
  signals.py                 FAVORITE / ARB / MOVER detection, run_scan()
  sizing.py                   2%-max-loss cap + quarter-Kelly sizing
  fees.py                     Kalshi taker fee approximation
  db.py                       SQLite schema + CRUD
  cli.py                      scan / take / resolve / pnl subcommands
tests/
  test_scanner.py            smoke test, both APIs mocked
```
