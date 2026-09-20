# NEPSE API — Add-on Feature

Self-contained Nepal Stock Exchange data system. Drop-in backend for any site
(e.g. a NEPSE screener page): run it, point your frontend at it, data stays
fresh via the twice-daily GitHub Action.

## Parts (all compile-checked)

| File | Role |
|---|---|
| `server.py` | REST API (`:8000`) — Summary, LiveMarket, SecurityList, SectorScrips, validation |
| `socketServer.py` | WebSocket stream (`:5555`) |
| `mcp_server.py` | MCP server for AI clients (`:8080`) |
| `updateStocksMap.py` | Daily updater → `stockmap.json` (567 securities → 540 mapped) |
| `stockmap.json` | Symbol → sector map, auto-committed twice daily |
| `stock_update.log` | Last run log, auto-committed with the map |
| `.github/workflows/update-stock-map.yml` | Schedule `0 0,12 * * *` + manual dispatch |

## Run locally (Windows)

```bat
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
python start_servers.py
```

Health: `http://localhost:8000/health` → `{"status":"ok"}` (expect 200).

## Daily auto-update chain (verified live)

`schedule` → checkout → setup-python 3.11 → pip install →
start `server.py` → `updateStocksMap.py` (SecurityList + SectorScrips) →
commit `stockmap.json` + `stock_update.log` → push.
Failures open a `Stock Map Update Failed` issue with both logs attached.

Proven: run #12 `Success in 37s`, auto-commit `b69178e`.

## Links this system connects to

- NEPSE via `nepse @ git+https://github.com/basic-bgnr/NepseUnofficialApi.git@dev`
- Internal: `:8000/health`, `/SecurityList`, `/SectorScrips`
- Upstream reference (third-party, currently 522): `https://nepseapi.surajrimal.dev`

## Recommendation

Pin the `nepse@dev` dependency to a commit SHA to remove the one
floating external link. Say the word and it gets pinned + pushed.
