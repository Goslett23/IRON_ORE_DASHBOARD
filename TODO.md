# Outstanding items before this dashboard can publish

This scaffold was built from `iron_ore_source_tracker.xlsx` (Mine candidates + Source order &
scoring sheets) and a confirmed `source_registry.json`. Where I had real sourced data, I used
it. Where I didn't, I marked the field `"TODO"` or omitted it rather than inventing a
plausible-looking number. This file collects everything that still needs a real analyst pass.

## Will currently fail `python automation/validate_data.py`, by design

- **`disruptions` is empty.** The xlsx is a source-monitoring candidate list, not an incident
  report — none of the 8 mines/hubs have a confirmed current disruption in the data I was
  given. `validate_data.py` requires at least one disruption entry to publish. Do not fill this
  with a placeholder incident; add a real one once a Tier 1/2 source confirms something is
  actually happening.
- **`reference_mines.items` has no coordinates.** The xlsx's Mine candidates sheet has no
  lat/lon column. All 8 entries are missing `lat`/`lon` (the keys are omitted, not null — a
  `null` value would crash the validator with a Python `TypeError` on the range comparison
  instead of a clean error message, since `validate_data.py`'s fallback default only applies
  when the key is absent). Needs real coordinates per mine/hub before the map will render
  anything besides an empty layer.
- **All four `kpis` values are `"TODO"`.** In particular "Disruption risk" can't be computed
  honestly until `disruptions` has real entries.
- **`tightness.indicators` is empty**, and `tightness.rating`/`assessment` are placeholders.
- **`outlook.scenarios.*` have empty `years`/`supply`/`demand` arrays** and no `assumptions`.
  Needs real REQ/IBRAM-sourced (or internal, clearly labelled) forecast numbers.
- **`drivers.end_use` shares are all missing** (share sums to 0, not 100). The four category
  names I used (Construction & infrastructure, Automotive & transport, Machinery & industrial
  equipment, Appliances & other manufacturing) are a reasonable generic steel end-use
  breakdown, not sourced yet — confirm the actual category split from World Steel Association
  before publishing, and adjust category names if their breakdown differs.
- **`drivers.items` evidence scores are all missing.** The four driver names (China
  property/construction cycle, global crude steel production, green steel/EAF transition,
  infrastructure & grid build-out) are plausible categories mirroring copper's driver
  structure, not sourced confidence scores.

## Registry gaps

- ~~`automation/source_registry.json`'s `primary` list has no entry for ArcelorMittal
  Liberia~~ — **Fixed.** Added as `arcelormittal-liberia-sec-6k` (SEC EDGAR 6-K, CIK 1243429),
  verified via the Q1 2026 filing to report Liberia iron ore production/shipments quarterly in
  the Mining segment table (better disclosure than the xlsx originally assumed — it's quarterly,
  not annual-only). `reference_mines.items` and `source_register` in `current.json` have been
  updated to match; this also answers the "disclosure cadence" open item below.
- Per the xlsx's own "Open verification items" list, still unconfirmed:
  - Exact PDF/press-release URL patterns for Vale, Rio Tinto, BHP, Fortescue, Anglo American
    and Champion Iron (each operator's IR site structure differs; the registry intentionally
    tracks index pages, not constructed URLs, per the architecture decision already made).
  - Whether BHP's Excel data book (attached to each Operational Review) has a stable,
    programmatically-pullable filename pattern — the current fetcher only handles HTML/JSON,
    not Excel.

## Attributable-share production

Architecture decision on record: production should be reported attributable-share, with the
100%-basis figure as a secondary field (mirroring FCX's Morenci at 72%, Cerro Verde at 55.66%
on the copper dashboard). The xlsx gives only a single "Est. 2025 production (Mt)" figure per
mine/hub with no ownership percentage, so none of the 8 `reference_mines.items` entries have
been split into attributable vs 100%-basis yet — each carries a `production_basis: "TODO"`
note flagging this. Needs the actual equity-interest percentages per mine/hub (e.g. Hope Downs
50%, IOC 59%, per the architecture-decision examples already given) before this can be done
correctly.

## No confirmed outlook forecaster for Canada

REQ (Australia, `aus-req-outlook`) and IBRAM (Brazil, `ibram-brazil-outlook`) are now in
`automation/source_registry.json` as genuine outlook forecasters, and `validate_data.py`
accepts `"REQ"` or `"IBRAM"` in a scenario note as provenance. A Canadian entry
(`nrcan-canada-stats`) was also added to the registry, but after checking bank research
(RBC/TD/Deutsche Bank) it was confirmed there is no genuine ongoing Canadian outlook
publication comparable to REQ or IBRAM — NRCan is predominantly historical/statistical, not a
forecast. It is registered as Tier 2 context only. `validate_data.py` deliberately does **not**
accept `"NRCan"` as a provenance citation. Outlook scenario notes touching Canada-specific
assumptions (Bloom Lake / Champion Iron) should keep citing `"internal"` until a real
forecaster is found — do not substitute NRCan, REQ, or IBRAM for a Canadian assumption.

## Known code issue carried over from the copper dashboard, not fixed here

`app.js`'s `renderKpis()` only has special-case styling for `tone: "red"` and `tone: "green"`;
anything else (including a hypothetical `tone: "orange"`) falls through to blue, and there's no
`.tone-orange` CSS class in `styles.css`. This scaffold avoids the issue by only ever using
`"blue"` as the placeholder tone — flagging it here rather than silently fixing app.js, since
the instruction was to reuse it, not rewrite it. Worth a real fix (or a deliberate decision not
to use an orange/amber tone at all) before analysts start setting real KPI tones.

## Deliberate design choices made while building this scaffold (not silent guesses)

- **China's domestic aggregate (xlsx row 9)** is documented in `reference_mines.note` and
  `methodology`, and listed under Tier 2 sources (USGS), but is **not** a `reference_mines.items`
  entry — it has no single mine location to plot, so forcing it into a lat/lon-keyed list
  would mean inventing a point. If you want it on the map anyway (e.g. at a representative
  coordinate), that's a call for you to make, not one I made silently.
- **`app.js`'s `renderDrivers()` chart-label shortener** (the hardcoded `<br>`-wrapping lookup
  for specific copper end-use names) was replaced with a plain identity function, since the
  placeholder end-use category names here don't match copper's and I don't know what the real
  final category names/line-wrapping needs will be. Revisit once `drivers.end_use` names are
  finalized.
- **Null-safety was added** to `renderDrivers()`'s share/evidence rendering (`item.share == null
  ? "" : ...`, `item.evidence == null ? "—" : ...`) so the TODO-heavy scaffold renders as blank
  rather than literal `"undefinedundefined%"` text. This is a real (small) code change beyond
  rebranding, made necessary by shipping a scaffold with intentionally incomplete data.
- **The "ICSG context" external map link** in copper's `index.html` was removed rather than
  replaced, since no equivalent iron-ore facility-map URL was provided.
