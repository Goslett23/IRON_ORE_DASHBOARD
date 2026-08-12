# Iron Ore Market Intelligence

A source-first weekly dashboard covering:

1. Mine disruption tracker
2. Current market tightness
3. Two-to-three-year supply/demand outlook
4. Cyclical and structural demand drivers

This repo replicates the architecture of TS Lombard's copper dashboard for iron ore. It is a
sibling project, not a fork or branch — the two commodities are tracked independently, though
the automation and validation code is deliberately kept structurally identical so fixes and
improvements can be ported between them by hand.

**This repo is currently a scaffold, not a published dashboard.** `data/current.json` matches
the schema exactly but most content fields are marked `"TODO"` or omitted rather than filled
with invented numbers. `python automation/validate_data.py` will fail until an analyst has
reviewed real sources and populated the required fields — that failure is intentional; see
[TODO.md](TODO.md) for the full list of what's outstanding.

## Interface

The dashboard uses the TS Lombard house style:

- Roboto for all interface and chart typography.
- Plotly.js for the mine map and every analytical chart.
- The approved TS Lombard blue, red and green ramps on a true-white canvas.
- Progressive disclosure for longer assessments, methodology and the source register.
- Grey reference markers for tracked major iron ore operations, separate from live disruption
  assessments.

Plotly.js is loaded from a pinned CDN version and Roboto from Google Fonts when the static site
opens. The underlying research data and HTML table remain available if a chart resource cannot
load.

## How this differs from the copper dashboard

- **No refined-market stage.** Copper's "concentrate vs refined" tightness framing assumes a
  smelting/refining step between mine and end use. Iron ore has no equivalent — ore goes
  roughly straight to steel mills (via beneficiation/pelletizing, not refining). The
  `tightness` section's `assessment` field carries a note to this effect; do not reuse copper's
  "refined balance" language without rethinking what the two-speed framing should mean here
  (e.g. mine-level supply vs seaborne/port availability, fines vs pellet premium).
- **Outlook provenance check.** Copper's `validate_data.py` requires every outlook scenario
  note to disclose either "internal" or "ICSG" (copper's forecaster). No iron ore source in
  `automation/source_registry.json` publishes an equivalent supply/demand outlook forecast, so
  this dashboard's validator instead accepts `"internal"`, `"REQ"` (Resources and Energy
  Quarterly, Australia) or `"IBRAM"` (Instituto Brasileiro de Mineração, Brazil). There is no
  confirmed equivalent for Canada — Canada-specific scenario assumptions (e.g. Bloom Lake)
  should cite `"internal"` until a real source is found.
- **Attributable vs 100%-basis production.** Where a mine/hub is jointly owned or reported at
  hub level (e.g. Rio Tinto's Pilbara sub-hubs), production should ultimately be reported on an
  attributable-share basis with the 100%-basis figure kept as a secondary field, mirroring how
  the copper dashboard handles FCX's Morenci (72%) and Cerro Verde (55.66%). The current
  scaffold has **not** made this adjustment — the `production_2025_est_mt` figures in
  `reference_mines.items` are hub/system-level totals from the source tracker, each flagged
  with a `production_basis: "TODO"` note. See TODO.md.

## Operating model

The weekly pipeline deliberately separates collection from judgment:

1. `automation/update_weekly.py` checks Tier 1 operator and regulatory sources.
2. It checks Tier 2 market and structural sources.
3. Only then does it query GDELT for wide-web discovery.
4. It writes `data/source_checks.json` and `data/candidates.json`, preserving URLs, source tier,
   timestamps and fingerprints.
5. It updates the dashboard's freshness metadata.
6. `automation/validate_data.py` blocks malformed data or unlabeled scenarios.
7. Material risk, tightness or outlook changes remain analyst-reviewed edits to
   `data/current.json`.

This avoids an automated news headline silently becoming an investment conclusion.

## Weekly schedule

`.github/workflows/weekly-iron-ore-update.yml` runs weekly and can also be started manually. It
commits the evidence snapshot, creating a versioned audit trail. The separate Pages workflow
validates, builds and deploys the site after each data change.

## Publish a shareable link

1. Push this project to the `main` branch of a GitHub repository.
2. In GitHub, open **Settings → Pages**.
3. Under **Build and deployment**, select **GitHub Actions**.
4. Run **Deploy iron ore dashboard** once from the Actions tab.
5. The resulting URL will be `https://YOUR-USERNAME.github.io/YOUR-REPOSITORY/` unless your
   organisation uses a custom GitHub Pages domain.

GitHub Pages sites are public by default. For confidential internal research, use an
access-controlled host instead of Pages. Note that `build_site.py` publishes the entire `data/`
directory, including `candidates.json` and `source_checks.json` — the same consideration
applies here as on the copper dashboard.

## Local preview

From the repository root:

```powershell
python automation/validate_data.py
python automation/build_site.py
python -m http.server 8000 --directory dist
```

Open `http://localhost:8000`. Until the scaffold is filled in (see TODO.md), the first command
will fail with a list of missing fields — that's the publication gate working as intended.

## Weekly analyst checklist

- Review every Tier 1 changed-source candidate before Tier 3 news.
- Confirm production bases: attributable vs 100%, hub-level vs single-mine, production vs sales.
- Do not sum operator impact estimates unless their units and baselines are comparable.
- Separate mine-level tightness from seaborne/port-level availability.
- Label internal scenarios and assumptions explicitly ("internal", "REQ" or "IBRAM").
- Move reviewed candidates into `data/current.json`, update the narrative, and validate before
  publishing.

## Files

- `data/current.json`: published dashboard dataset and evidence (currently a TODO-heavy
  scaffold — see TODO.md).
- `data/source_checks.json`: automated check health and fingerprints (empty until the first
  automation run).
- `data/candidates.json`: unreviewed source-change and broad-news queue (empty until the first
  automation run).
- `automation/source_registry.json`: ordered source registry.
- `automation/update_weekly.py`: collector and audit writer (unchanged from the copper
  dashboard — source-agnostic).
- `automation/validate_data.py`: publication gate (one line changed from copper's version — see
  "How this differs" above).
- `automation/build_site.py`: dependency-free static build (unchanged from the copper
  dashboard).
- `TODO.md`: everything that still needs real analyst/sourced input before this dashboard can
  pass validation and publish.
