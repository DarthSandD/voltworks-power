# VoltWorks Power

Live site: https://darthsandd.github.io/voltworks-power/ (GitHub Pages, serves `index.html`).

Single-file electrical calculator suite: system base (per-unit), feeder voltage
drop, transformer sizing + fault level, LV cable sizing, short-circuit at bus,
design-standard selector (IEC / ANSI / IEEE), live load-study slider, SLD canvas
builder, and a generated engineering report. No build step, no backend, no deps.

## Input persistence

Every input (numbers, selects, the load slider) autosaves to the site's own
`localStorage` (key `voltworks-power-v1`, debounced 300ms) and restores before
the first calculation on every load — refresh-safe, no account, no backend.
The ↩ Reset button in the header clears saved state back to defaults.

## SLD persistence

The single-line diagram canvas (components, wires, breaker/load state) autosaves
to `localStorage` (key `voltworks-sld-v1`) on every change — place, wire, operate,
delete, clear, and drag — and restores on load.

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

- **Correctness pass** — transformer fault current now derives from the
  transformer's own kVA/%Z/secondary kV (was using the 100 MVA system base,
  ~3.6× too high and independent of rating); report generation no longer throws
  under the default IEC fault standard (it referenced an ANSI-only field);
  SLD canvas persists across refresh.
