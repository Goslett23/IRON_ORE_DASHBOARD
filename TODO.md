## August 2026: expanded reference-mine coverage (5 new operators added)

`reference_mines.items` covered only 6 mines/hubs, all sourced from the original xlsx candidate
list, and left real gaps: Kumba (South Africa), LKAB (Sweden), Iron Ore Company of Canada, and
Samarco (Brazil) were all significant producers with no tracking, and India — a top-tier global
producer/exporter via NMDC — was absent from the registry entirely. Five new rows were added to
`reference_mines.items`, each backed by an operator or regulated-filer primary source, with a
matching entry added to `automation/source_registry.json`'s `primary` list and to
`source_register` in `current.json` so the weekly job actually checks them going forward:

- **Kumba Iron Ore (Sishen + Kolomela), South Africa.** FY2025 production 36.1 Mt (+1% y/y), via
  Kumba's own Q4/FY2025 production and sales report and trading statement. Kumba is a distinct
  JSE-listed, majority Anglo American-owned entity from Anglo American's Minas-Rio (Brazil) —
  confirm this wasn't an oversight elsewhere before assuming "Anglo American" coverage was
  complete.
- **LKAB (Kiruna + Malmberget/Svappavaara), Sweden.** ~26 Mt of iron ore products delivered in
  2025 per LKAB's Year-end Report 2025. LKAB supplies roughly 86% of the EU's iron ore — the
  single largest gap closed in this pass. Note the confirmed figure is *deliveries*, not
  mine-head production; needs reconciling against how other rows define their figure.
- **Iron Ore Company of Canada (IOC), Labrador City.** Was already inside the existing Rio Tinto
  SEC 6-K registry entry's stated scope ("Pilbara, IOC and Simandou") but had no
  `reference_mines.items` row of its own — a genuine gap between what the registry claimed to
  track and what was actually in the published dataset. Figure used (16.5 Mt) is the *lower end
  of FY2025 guidance*, not a confirmed actual.
- **Samarco (Germano Complex), Brazil.** Vale/BHP 50:50 joint venture restarting post-2015
  Fundão dam disaster; targeting 15 Mt of pellets/fines in 2025 (~60% of pre-disaster capacity).
  Material given the ~US$28bn combined Vale/BHP disaster settlement with Brazil signed in 2025.
  Reliability marked Medium — Vale/BHP corroborate Samarco's own target but don't always restate
  it on a fixed schedule; needs a confirmed direct publication cadence from Samarco itself.
- **NMDC — Bailadila Complex (Kirandul/Bacheli), India.** India was completely absent from this
  dashboard despite NMDC's FY2025-26 record production of 53.15 Mt (+21% y/y, state-owned).
  `production_2025_est_mt` is left as a `"TODO"` string rather than a number: NMDC's own
  releases give a company-wide total across Bailadila and its separate Donimalai (Karnataka)
  mine, and a Bailadila-only split was not found in this pass — do not silently split the
  53.15 Mt total or invent a per-site number.

### Still not added — investigated and deliberately left out

- **Ukraine (ArcelorMittal Kryvyi Rih / Metinvest).** Real and currently newsworthy: AMKR was
  reported operating at roughly 75% of pre-war capacity (~7.5 Mt/year concentrate) with
  recurring shutdowns from both power-grid attacks and, as of May 2026, a logistics dispute with
  Ukrzaliznytsia (national rail) that halted mining entirely for a period. **Not added** to
  either `disruptions` or `reference_mines.items` in this pass because every source found
  (SteelOrbis, Interfax-Ukraine, GMK Center) is Tier 3 news reporting — no ArcelorMittal IR
  release, SEC 6-K exhibit, or Metinvest primary disclosure confirming current output was
  located. Per this dashboard's own methodology, Tier 3 items are for flagging candidates
  pending primary confirmation, not for publishing as a confirmed disruption or reference row.
  Metinvest itself (privately held, Rinat Akhmetov/SCM) does not file with the SEC and has no
  confirmed public production-disclosure cadence — worth checking whether it publishes its own
  operational updates before the next pass. If a Tier 1/2 source is found, this is a strong
  candidate for `disruptions` (not `reference_mines`), given the active, evolving nature of the
  situation.
- **NMDC — Donimalai, India.** NMDC's second complex (Karnataka). Not given its own
  `reference_mines.items` row in this pass — see the Bailadila entry above.
- Other mines identified but not researched in this pass, for a future analyst to assess:
  Assmang (Khumani/Beeshoek, South Africa — Assore/African Rainbow Minerals JV), Sino Iron and
  Karara (Australian magnetite, both China-linked JVs), SNIM (Mauritania), Shougang Hierro Perú
  (Marcona, Peru), and the US Mesabi Range (Minnesota — Cleveland-Cliffs/US Steel). None of
  these were checked for a confirmed Tier 1/2 source or coordinates — do not assume they were
  vetted and rejected; they simply weren't reached.

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
