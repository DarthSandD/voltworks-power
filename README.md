# VoltWorks Power

Live site: https://darthsandd.github.io/voltworks-power/ (GitHub Pages, serves `index.html`).

Single-file electrical calculator suite for power-systems engineers: system base
(per-unit), feeder voltage drop, transformer sizing + fault level, LV cable sizing,
short-circuit at bus, design-standard selector (IEC / ANSI / IEEE), Newton–Raphson
power flow, motor-starting voltage dip, cable short-circuit withstand, power-factor
correction, arc-flash hazard (IEEE 1584-2018), live load-study slider, SLD canvas
builder, and a generated engineering report. No build step, no backend, no deps.

## Modules

| # | Module | What it does |
|---|--------|--------------|
| 01 | **Power Systems** | Per-unit base, feeder VD, transformer sizing + fault level, LV cable sizing, short-circuit at bus |
| 02 | **Power Flow** | Newton–Raphson AC load flow — bus Vm/angles, branch flows, losses, loading. Slack/PV/PQ buses; add/remove buses + branches |
| 03 | **Motor Starting** | Locked-rotor inrush + PCC voltage dip (impedance divider, IEC 60034-12). Compares DOL / star-delta / soft-starter / VFD |
| 04 | **Cable SC Withstand** | Adiabatic `k²S² ≥ I²t` (IEC 60364-4-43): minimum area + max clearing time |
| 05 | **PF Correction** | Capacitor sizing `Q = P(tanφ₁−tanφ₂)`, before/after kVA · kVar · current |
| 06 | **Arc Flash** | Incident energy + arc-flash boundary, IEEE 1584-2018 empirical model: arcing current (full + reduced case), enclosure size correction, NFPA 70E PPE category. Equipment presets; 208 V–15 kV |
| 07 | **Network Analysis** | Bus results, branch flows, short-circuit, compliance and dispatch tables |
| 08 | **SLD Builder** | Place / wire / operate single-line diagram, energisation tracing, export SVG |
| 09 | **Engineering Report** | Collects every module's latest result into one printable report |

## Input persistence

Every input (numbers, selects, the load slider) autosaves to the site's own
`localStorage` (key `voltworks-power-v1`, debounced 300ms) and restores before
the first calculation on every load — refresh-safe, no account, no backend.
The ↩ Reset button in the header clears saved state back to defaults.

## SLD persistence

The single-line diagram canvas (components, wires, breaker/load state) autosaves
to `localStorage` (key `voltworks-sld-v1`) on every change — place, wire, operate,
delete, clear, and drag — and restores on load.

## Power-flow persistence

The load-flow network (buses + branches) autosaves to `localStorage`
(key `voltworks-flow-v1`) and restores on load.

## Verification

- **Load flow** validated against `pandapower` (Newton–Raphson): bus Vm, angles and
  total losses match to 4 decimal places on the demo network.
- **Arc flash** validated against the `liaungyip/arcflash` IEEE 1584-2018 reference
  implementation on both Annex D worked examples (MV 4.16 kV and LV 480 V): arcing
  current, incident energy and arc-flash boundary match to 4 decimal places for
  both the full and reduced-current cases.
- Transformer fault current, cable SC withstand and PF sizing checked against
  hand-computed values.


## Fault-current convention

- **Transformer card** reports the un-factored symmetrical LV fault current:
  `Isc = kVA / (√3 · Vsec · (%Z/100))`.
- **Short-circuit card** applies the IEC 60909 voltage factor `c = 1.10`
  (`Ik″ = c · Isc`) plus peak `Ip ≈ 2.55 · Ik″` (κ = 1.8, LV). Under ANSI it
  reports symmetrical `Ik` plus momentary duty `×1.15`.
- The two cards therefore differ by the `c` factor by design (e.g. 28.87 kA
  un-factored vs 31.75 kA IEC for 1000 kVA / 5% / 0.4 kV).

## Deploy

Local `main` pushes to remote `main` (`git push origin main`).
Pages rebuilds in ~1–2 min — verify live with:
`curl -s https://darthsandd.github.io/voltworks-power/ | grep -c vwRestore` (expect 2).

## Changelog

- **Arc Flash (IEEE 1584-2018)** — new module: arcing current (full + reduced),
  enclosure size correction, incident energy, arc-flash boundary and NFPA 70E PPE
  category, with equipment presets. Validated against the `liaungyip/arcflash`
  reference implementation on the IEEE Annex D MV and LV examples (4 dp match).
- **Modules added** — Power Flow (Newton–Raphson), Motor Starting, Cable SC Withstand,
  PF Correction. Load flow verified against `pandapower`.
- **Correctness pass** — transformer fault current now derives from the
  transformer's own kVA/%Z/secondary kV (was using the 100 MVA system base,
  ~3.6× too high and independent of rating); report generation no longer throws
  under the default IEC fault standard (it referenced an ANSI-only field);
  SLD canvas persists across refresh.
