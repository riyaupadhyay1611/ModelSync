# ModelSync

**DOE-2.2 SIM → Excel conversion tool.** A single-file browser app that parses EnergyPlus/DOE-2.2 simulation output files and generates a formatted Excel workbook — no server, no install, no file uploads.

---

## What it does

Drop in DOE-2.2 output files (Baseline + Proposed), configure utility rates, and click **Generate Excel**. ModelSync reads every major simulation report and produces a `.xlsx` with two tabs — `BL Data` and `Proposed Data` — sharing the same 115+ column schema.

### Report sections parsed

| Report | Data extracted |
|--------|---------------|
| **BEPS / BEPU** | Energy end-uses by category (kBtu + kWh), EUI, conditioned area |
| **LV-D** | Envelope summary — per-orientation window/wall areas, WWR, weighted U-values, roof |
| **LV-E** | Underground / slab-on-grade U-values |
| **LV-B** | Space inventory — conditioned area, occupancy, LPD, EPD |
| **SV-A** | Supply/return fan kW, supply CFM, OA CFM |
| **SS-A** | Per-system peak heating/cooling loads (summed → building peak) |
| **PS-B** | Annual kWh + therms, peak electric kW + peak hour |
| **SS-R / PS-A** | Unmet hours, plant rollup cross-checks |
| **INP** | Glass properties, chiller/boiler COP, utility rates, shading |
| **EPW** | Climate zone validation, design temperatures, lat/long |

---

## Usage

1. Open `ModelSync.html` in any modern browser (Chrome, Edge, Firefox).
2. Drop files into the upload zones:
   - **Baseline** — `.SIM` required, `.INP` optional
   - **Proposed** — `.SIM` required, `.INP` optional
   - **EPW** — optional, enables climate zone + lat/long extraction
3. Fill in the **Configuration** section:
   - Climate zone, output file name, auto-pair toggle
   - Electricity rate ($/kWh), gas rate ($/therm), carbon factors
4. Optionally enter coordinates and an NREL API key to auto-fetch utility rates from the [OpenEI URDB](https://developer.nrel.gov/signup/).
5. Click **Generate Excel** — the `.xlsx` downloads immediately.

---

## File support

| Extension | Role |
|-----------|------|
| `.sim` / `.SIM` | DOE-2.2 simulation output — primary data source |
| `.inp` / `.INP` | DOE-2.2 input deck — glass, COP, shading, rates |
| `.epw` | EnergyPlus Weather file — climate zone, design temps, coordinates |

Multiple baseline and proposed SIM/INP files can be loaded at once; ModelSync pairs them automatically when **Auto-pair** is enabled.

---

## Output

A single `.xlsx` file with:

- **BL Data tab** — all extracted and derived columns for the Baseline model
- **Proposed Data tab** — same schema for the Proposed model
- **115+ columns** covering energy end-uses, envelope, HVAC, lighting, peak loads, rates, carbon, and derived metrics (savings, % improvement, EUI delta)

---

## Privacy

Everything runs in the browser. Files are read locally via the File API and never sent anywhere. No backend, no cookies, no telemetry.

---

## Tech

- Vanilla HTML + CSS + JavaScript — zero dependencies, zero build step
- [`xlsx-js-style`](https://github.com/gitbrent/xlsx-js-style) (bundled) — Excel workbook generation with cell styling
- NREL URDB API — optional, for utility rate lookup by location

---

## Development

The entire app lives in `ModelSync.html`. Structure:

```
<style>          — UI (lilac dark theme, Space Grotesk / Inter / JetBrains Mono)
<body>           — Layout: fixed header + labeled sidebar + main + right panel
<script>
  SIMParser      — Parses .sim report text into structured row objects
  INPParser      — Parses .inp input deck for supplemental fields
  nrelUrdbLookup — Async NREL API fetch
  COLUMNS[]      — 115+ column definitions (key, label, format, formula)
  enrichRow()    — Derives calculated fields from parsed data
  buildWorkbook()— Assembles the xlsx workbook via xlsx-js-style
  UI wiring      — Dropzone binding, chip rendering, run/clear logic, toasts
```

To modify the output schema, edit the `COLUMNS` array. Each entry is:

```js
{ key: 'eui_net', label: 'Net EUI (kBtu/sf·yr)', fmt: '#,##0.0', fn: r => r.eui_net }
```

---

## Requirements

- A modern browser with ES2020 support (Chrome 90+, Firefox 90+, Edge 90+)
- No local server needed — open the HTML file directly
