# Tickerboard

A live watchlist, portfolio tracker and practice-trading desk, built as a single HTML page that runs as a Claude artifact.

- **Portfolio mode:** add any listed company, fund, index, currency or crypto, track intraday and historical charts, and record your real holdings to see value and gain in pounds.
- **Practice mode:** a separate pretend-money account. Buy and sell at live prices, backdate a buy to a past closing price, and compare your return with the FTSE 100.

## Files

| Path | What it is |
| --- | --- |
| `index.html` | The whole app: markup, styles and script in one file |
| `n8n/tickerboard-price-feed.json` | Export of the n8n workflow that supplies prices |

## How prices reach the page

The page runs inside Claude and calls the n8n connector, which runs the **Tickerboard · Price Feed** workflow on `covase.app.n8n.cloud`. That workflow fetches from Yahoo Finance's public chart API and returns compact JSON.

The page calls two n8n tools:

1. `execute_workflow` with trigger `Price Request` and a webhook-style `query`
2. `get_workflow_execution`, polled until the run succeeds, reading the `Build Result` node's output

### Feed query parameters

| Parameter | Values | Notes |
| --- | --- | --- |
| `mode` | `quotes`, `chart`, `search` | |
| `symbols` | Comma-separated Yahoo tickers, up to 40 | London shares end in `.L`, e.g. `TSCO.L`; FX pairs like `USDGBP=X` |
| `range` | `1d`, `5d`, `1mo`, `6mo`, `ytd`, `1y`, `5y`, `max` | Interval is chosen per range |
| `from`, `to` | Unix seconds, window up to 400 days | Daily closes; used for backdated practice buys |
| `q` | Search text | `mode=search` only |

Each result carries `requested`, `symbol`, `name`, `currency`, `exchange`, `type`, `price`, `previousClose`, `chartPreviousClose`, day and 52-week ranges, `volume`, `marketTime`, `session` and `points` (`[unixSeconds, close]`).

## Storage

Inside Claude, the watchlist (`data/users/<id>/board`) and practice account (`data/users/<id>/practice`) are saved to each viewer's private artifact store. Opened anywhere else, the page falls back to the browser's local storage.

## Running it outside Claude

Opening `index.html` directly works, but shows no prices: the n8n connector is only available inside Claude. This repo is the source and a backup, not a hosted copy.

## Data notes

- Yahoo Finance's chart API is unofficial and can change without notice. Treat figures as indicative.
- London quotes arrive about 15 minutes behind the market. Every line shows its last trade time.
- London prices are quoted in pence (`GBp`) and converted to pounds for values.
- Positions in other currencies are converted at the current exchange rate, including their cost, so gains on them are estimates.
- Practice mode pools cost at an average price per holding, as HMRC's Section 104 pool does. It can charge 0.5% stamp duty on London share purchases (ETFs exempt; AIM shares can't be detected) and an optional flat dealing fee. Dividends, spreads and FX charges aren't modelled.

## Updating the published artifact

The artifact is republished from `index.html`. If you edit the file, ask Claude to republish it to the existing Tickerboard artifact so the link stays the same.
