# Costco Gas Tracker

Fetches Costco gas prices for tracked ZIP codes and serves them as a static PWA.

## Layout

- `fetch_data.py` — fetches prices, writes `pwa/data/*.json`.
- `config.json` — ZIP codes to track.
- `zip_cache.json` — ZIP → lat/lng/label cache (avoids re-geocoding).
- `pwa/` — the static PWA (served as-is by GitHub Pages).
- `costco_gas_prices.py` / `costco_gas_prices.ipynb` — ad-hoc CLI/notebook lookups.
- `run_local.sh` — local wrapper: pulls latest, runs the fetcher, commits +
  pushes data changes if any. Logs to `~/Library/Logs/costco_gas.log`.
- `launchd-disabled/com.user.costco-gas.plist` — local launchd schedule.
  **Currently disabled** — GitHub Actions (`.github/workflows/pages.yml`)
  handles the hourly fetch + deploy instead. Reload it with
  `cp launchd-disabled/com.user.costco-gas.plist ~/Library/LaunchAgents/ && launchctl load ~/Library/LaunchAgents/com.user.costco-gas.plist`
  if you ever want a local schedule again.

## Deployment

GitHub Actions fetches data hourly and deploys `pwa/` directly to GitHub
Pages for this repo. See `.github/workflows/pages.yml`.

## Local dev / manual runs

```
python3 -m venv .venv
.venv/bin/pip install "curl_cffi>=0.7"
.venv/bin/python fetch_data.py
```

`run_local.sh` mirrors the GitHub Actions job for local/manual runs: it pulls
latest, runs `fetch_data.py`, and commits + pushes `pwa/data` and
`zip_cache.json` if they changed. It expects `.venv/bin/python` to exist.

One thing to watch if you push from a local script — SSH push needs your key
available. If it's in the macOS Keychain (`UseKeychain yes` in `~/.ssh/config`
and `ssh-add --apple-use-keychain`), `git push` will work. Otherwise add to
`~/.ssh/config`:

```
Host github.com
  UseKeychain yes
  AddKeysToAgent yes
  IdentityFile ~/.ssh/id_ed25519
```

then run `ssh-add --apple-use-keychain ~/.ssh/id_ed25519` once.
