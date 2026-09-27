# VoltWorks Power

Live site: https://darthsandd.github.io/voltworks-power/ (GitHub Pages, serves `index.html`).

Single-file electrical calculator suite for power-systems engineers: system base
(per-unit), feeder voltage drop, transformer sizing + fault level, LV cable sizing,
short-circuit at bus, design-standard selector (IEC / ANSI / IEEE), Newton–Raphson
power flow, motor-starting voltage dip, cable short-circuit withstand, power-factor
correction, arc-flash hazard (IEEE 1584-2018), live load-study slider, SLD canvas
builder, and a generated engineering report. No build step, no backend, no deps.

**Beginner-friendly:** every section has a collapsible **“💡 How this works”**
panel — a plain-English explanation, a step-by-step guide, a table defining each
input, rules of thumb, and a **one-click worked example** that fills the inputs and
runs the calculation. A header **“Explain everything”** button opens them all, and
a **📚 Glossary** modal defines every technical term.

## Design system

- **Colour psychology (semantic, not decorative):** one primary accent
  (**blue** `#4c8dff` dark / `#2563eb` light) for brand, actions and focus.
  **Amber = caution**, **red = danger**, **green = pass** — reserved strictly for
  meaning, never used as decoration, so a warning never looks like a logo.
- **Dual themes, both first-class:** dark and light each define their own full
  token set (`--bg`, `--surface`, `--border`, `--accent`, `--on-accent`, semantic
  colours, elevation, glow). Canvas + SVG colours are read from the CSS variables
  at runtime (`THEME()`), so the SLD and live-study chart re-theme correctly.
- **Accessibility:** all 17 text/background pairs meet **WCAG AA (≥4.5:1)**;
  `:focus-visible` rings on every interactive element; `prefers-reduced-motion`
  respected; `--on-accent` keeps button labels legible on accent fills in both themes.
- **Depth & polish:** layered shadows, tabular-nums on results, themed scrollbars,
  hover elevation on cards, custom text selection.

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
| 08 | **SLD Builder** | ETAP-style editor: real IEC/ANSI symbols, orthogonal auto-routed wires, grid + snap, multi-select (shift-click + marquee), undo/redo, zoom/pan/fit, per-element properties (tag, name, ratings, %Z), auto-tagging, energisation tracing, export SVG + PNG |
| 09 | **Engineering Report** | Collects every module's latest result into one printable report |

## Interactive help & worked examples

- Each of the 9 sections carries a **“💡 How this works”** panel: what the section
  does, numbered how-to steps, an input glossary table, rules of thumb, and a
  **⚡ Load this example & run it** button.
- Worked examples are pre-loaded with realistic values and immediately calculated:
  Power Systems (500 kW / 250 m / 1000 kVA), Motor Start (75 kW on 25 kA),
  Cable SC (25 kA, 0.2 s, 70 mm² Cu/XLPE), PF (500 kW 0.75→0.95),
  Arc Flash (IEEE Annex D LV case), Power Flow (4-bus demo), SLD (ready-made
  generator→breaker→bus→transformer→bus→motor+load diagram).
- **Explain everything** opens all panels at once; **📚 Glossary** opens a modal
  with 15 plain-English definitions (per-unit, bolted fault, incident energy,
  slack/PV/PQ bus, adiabatic check, …).

## Input persistence

Every input (numbers, selects, the load slider) autosaves to the site's own
`localStorage` (key `voltworks-power-v1`, debounced 300ms) and restores before
the first calculation on every load — refresh-safe, no account, no backend.
The ↩ Reset button in the header clears saved state back to defaults.

## SLD persistence

The single-line diagram canvas (components, wires, breaker/load state, tags and
properties) autosaves to `localStorage` (key `voltworks-sld-v1`) on every change —
place, wire, operate, drag, property edit, delete, clear — and restores on load.

## SLD editor (ETAP-style)

- **Symbols:** generator, transformer, breaker, busbar, motor, load — drawn as real
  IEC/ANSI glyphs (not text), theme-aware, with live energisation colouring.
- **Routing:** wires are orthogonal (manhattan) paths anchored to each symbol's
  connection point, previewed as a dashed ghost while you draw.
- **Editing:** drag to move (grid snap, 20 px); Shift-click or drag a marquee to
  multi-select and move/delete a group; **Ctrl+Z / Ctrl+Y** undo/redo (60-deep);
  **Del** deletes; **# Auto-tag** renumbers every element (G1, B2, T1, BUS1 …).
- **View:** wheel = zoom to cursor (0.3×–3×), Space or middle-drag = pan,
  ⤢ Fit frames everything, ▦ Grid and 🧲 Snap toggle.
- **Properties panel:** with one element selected, edit its tag, name, rating
  (kVA / kV), transformer %Z and breaker state, and see its energisation status.
- **Export:** ⬇ SVG (vector, for CAD) or ⬇ PNG (2× raster, for reports) — both
  render exactly what is on the canvas, in the active theme.

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

- **SLD builder rebuilt to ETAP grade (phase 2)** — real IEC/ANSI symbol glyphs;
  orthogonal auto-routed wires with a live ghost preview; grid + snap-to-grid;
  multi-select via shift-click and marquee, group move/delete; 60-step undo/redo
  (Ctrl+Z / Ctrl+Y); wheel zoom to cursor, pan (Space / middle-drag), fit-to-view;
  a per-element properties panel (tag, name, rating, transformer %Z, breaker
  state, energisation status); automatic tagging (G1/B2/T1/BUS1/M1/L1) with a
  renumber button; and SVG + PNG export that render the canvas as drawn.
- **Design system + UI/UX refresh (phase 1)** — one semantic accent (blue);
  amber/red/green reserved for caution/danger/pass; dual light+dark token sets;
  WCAG AA verified (17/17 pairs); focus-visible rings, reduced-motion support,
  themed scrollbars, layered elevation, tabular-nums results. Canvas/SVG colours
  now read live CSS variables so both themes re-theme correctly.
- **Beginner-friendly interactive layer** — “💡 How this works” panels on all 9
  sections (plain-English explanation + step-by-step + input glossary + rules of
  thumb), one-click worked examples that fill inputs and run the calc, a
  “Explain everything” toggle, a 📚 Glossary modal, and a ready-made SLD demo.
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
