# Worldbook

**A layered flow-map of the world on a dark-space globe** — 26 animated layers tracing how people, money, goods, and information move between countries, drawn as arcs on a MapLibre GL globe against a dark starfield.

**Live site:** https://worldbook.earth

The entire application is a single self-contained `index.html` (~1.8 MB). The flow geometry, all 242 country polygons, and the full UI are baked into that one file — **there is no build step required to run it.** Open it in a browser and it works.

---

## What it is

Worldbook renders a spinnable, zoomable globe (MapLibre GL JS v5.6.1) on a dark-space background. On top of the base map it draws **flow layers** — animated arcs between country pairs, each arc sized by a weight and animated with travelling dots to show direction. You pick one layer at a time from the thematic groups; hovering an arc reveals its endpoints, value, unit, and source.

Two data structures drive it, both inlined in `index.html`:

- **`FLOWS`** — the layer config object, one entry per layer: `label, group, unit, desc, color, dotColor, legend, note, sources[], edges[]`. Each edge is `{from:"ISO3", to:"ISO3", w:<number>}`, optionally `c:"#hex"` for per-edge colour (used by the multi-substance `drugs` layer). The `debtout` / `debtin` layers don't carry their own edges — they reuse the `debt` layer's edge store via `edgesRef:"debt"` + a direction flag.
- **`DATA`** — a GeoJSON FeatureCollection of 242 country polygons (the base geometry, not the flow store).

---

## Layer catalog

All 26 flow layers, grouped by their actual `group` value, with the real unit and cited sources pulled straight from `FLOWS`.

### Connections
| Layer | `key` | Unit | Sources |
|---|---|---|---|
| Remittance corridors | `remittances` | US$ billions / yr (approx) | World Bank Bilateral Remittance Matrix / KNOMAD; World Bank Migration & Development Brief 39 (2023) |
| Bilateral debt (creditor → debtor) | `debt` | US$ billions of debt stock (approx) | World Bank International Debt Statistics; AidData — Chinese overseas development finance |
| Migration corridors | `migration` | millions of people (foreign-born stock) | IOM World Migration Report 2024; UN DESA International Migrant Stock 2020 |
| Trade flows (top exports) | `trade` | US$ hundreds of billions (annual goods) | UN Comtrade; WTO World Trade Statistical Review |
| Submarine cables | `cables` | major intercontinental fibre-optic routes | TeleGeography — Submarine Cable Map |

### Resources
| Layer | `key` | Unit | Sources |
|---|---|---|---|
| Oil & gas | `oil` | major crude oil & LNG trade routes | IEA Oil Market Report; U.S. EIA Petroleum & Other Liquids |
| Food & grain | `food` | major cereal & staple-crop trade | FAO Food Outlook; USDA Foreign Agricultural Service |
| Minerals & metals | `minerals` | iron ore, copper, nickel, cobalt, rare earths | USGS Mineral Commodity Summaries |
| Timber & wood | `wood` | logs, lumber, pulp & wood products | FAO Forest Products Statistics |

### Debt
| Layer | `key` | Unit | Sources |
|---|---|---|---|
| Debt — money lent out | `debtout` | US$ billions of bilateral lending | World Bank International Debt Statistics; AidData |
| Debt — money owed | `debtin` | US$ billions of bilateral debt owed | World Bank International Debt Statistics; AidData |

*(`debtout` and `debtin` are lender-side and borrower-side views of the same `debt` edge store.)*

### Human movement
| Layer | `key` | Unit | Sources |
|---|---|---|---|
| Refugee flows | `refugees` | major forced-displacement corridors | UNHCR Global Trends / Refugee Data |
| International students | `students` | major international student corridors | UNESCO Institute for Statistics — Global Flow of Tertiary Students |

### Health
| Layer | `key` | Unit | Sources |
|---|---|---|---|
| Medical brain drain | `braindrain` | relative scale (foreign-trained doctors & nurses) | WHO Global Strategy on Human Resources for Health; OECD International Migration of Health Workers |

### Illicit flows
| Layer | `key` | Unit | Sources |
|---|---|---|---|
| Human trafficking | `trafficking` | relative corridor prominence (detected victims) | UNODC Global Report on Trafficking in Persons 2024 |
| Wildlife trafficking | `wildlife` | relative corridor prominence (seizures) | UNODC World Wildlife Crime Report; CITES trade & seizure data |
| Counterfeit goods | `counterfeit` | relative trade value (seizures) | OECD/EUIPO — Trade in Counterfeit and Pirated Goods |
| Drug routes | `drugs` | major trafficking corridors by substance | UNODC World Drug Report; EMCDDA European Drug Report |
| Arms trafficking | `arms` | major illicit small-arms & weapons flows | Small Arms Survey; UNODC Global Study on Firearms Trafficking |

### Infrastructure & travel
| Layer | `key` | Unit | Sources |
|---|---|---|---|
| Flight corridors | `flights` | busiest international air corridors | OAG / IATA air-traffic statistics |
| Shipping lanes | `shipping` | major maritime trade lanes & chokepoints | UNCTAD Review of Maritime Transport |

### Natural systems
| Layer | `key` | Unit | Sources |
|---|---|---|---|
| Ocean currents | `currents` | major surface currents (warm / cold) | NOAA — Ocean currents |
| Bird migration flyways | `flyways` | major global flyways | Convention on Migratory Species — Flyways |
| Saharan dust | `dust` | trans-Atlantic dust transport | NASA Earth Observatory — Saharan dust |

### Historic routes
| Layer | `key` | Unit | Sources |
|---|---|---|---|
| Silk Road | `silkroad` | overland + maritime routes (c. 130 BCE – 1450 CE) | UNESCO — Silk Roads Programme |
| Age of Exploration | `voyages` | landmark voyages, 1405–1522 | Britannica — Age of Discovery |

---

## Data & provenance

The governing principle of this project:

> **Every number traces to a named source; geometry is not fabricated; absence-by-omission is treated as a lie.**

Every layer carries a `sources[]` array, and every edge carries a weight. The primary data sources actually cited across the 26 layers:

- **Money & debt** — World Bank (Bilateral Remittance Matrix / KNOMAD, International Debt Statistics, Migration & Development Brief 39), AidData (Chinese overseas development finance)
- **Trade & commodities** — UN Comtrade, WTO World Trade Statistical Review, IEA Oil Market Report, U.S. EIA, FAO (Food Outlook, Forest Products Statistics), USDA Foreign Agricultural Service, USGS Mineral Commodity Summaries
- **People** — UN DESA International Migrant Stock, IOM World Migration Report 2024, UNHCR Global Trends, UNESCO Institute for Statistics, WHO, OECD (International Migration of Health Workers)
- **Illicit flows** — UNODC (World Drug Report, Trafficking in Persons 2024, World Wildlife Crime Report, Global Study on Firearms Trafficking), EMCDDA European Drug Report, OECD/EUIPO, CITES, Small Arms Survey
- **Infrastructure & natural systems** — TeleGeography, OAG / IATA, UNCTAD, NOAA, NASA Earth Observatory, Convention on Migratory Species, UNESCO Silk Roads, Britannica

---

## A note on what the arc widths mean

Arc width is **not** a single common unit across layers. Read it per layer:

- **Absolute-magnitude layers** carry real quantities in their stated unit — e.g. `migration` (millions of people, foreign-born stock), `remittances` / `debt` / `trade` (US$ billions), and the resource layers (relative trade volumes).
- **Relative-prominence layers** — `trafficking`, `wildlife`, `counterfeit`, and `braindrain` — encode **relative corridor prominence, not absolute counts.** A wider arc means a more prominent corridor in the underlying reports; it is **not** a victim count, a dollar figure, or a headcount. Each of these layers says so in its own `note`.
- **`cables` uses a uniform weight** (every corridor has `w:1`). For submarine cables the **width encodes nothing meaningful** — the layer shows *where* the intercontinental fibre routes run, not their capacity or traffic.

Don't read the four relative layers or the cables layer as if their widths were measured magnitudes.

---

## Running it

```bash
# clone
git clone https://github.com/never-nude/worldbook.git
cd worldbook

# run — just open the file
open index.html          # macOS
# or: xdg-open index.html # Linux
# or drag index.html into any modern browser
```

No build, no server, no `npm install`. Internet access is needed only at runtime for:

- **MapLibre GL JS v5.6.1** (loaded from the unpkg CDN),
- **map glyphs** (label fonts) and **raster tiles** — satellite and physical basemaps from ArcGIS World Imagery / World Physical, plus MapLibre demo glyphs.

All flow data and all 242 country polygons are baked into `index.html`, so the data itself needs no network.

---

## Data pipeline (current state — being migrated)

The layer data is generated and revised by a set of one-shot **Python scripts in the repo root**. Each one mutates `index.html` **in place** by string / regex surgery — locating a block in the HTML and rewriting it:

- **Layer generators** — `build_layers.py`, `commodities_flows.py` (oil/food/minerals/wood), `illicit_flows.py` (trafficking/wildlife/counterfeit/drugs/arms), `health_layers.py` (brain drain), `paths_layers.py` (currents/flyways/dust/silk road/voyages/flights/shipping), and the debt set (`debt_deepen.py`, `debt_split.py`, `debt_followmoney.py`, `debt_polish.py`).
- **Performance / quality passes** — `load_perf3.py`, `load_quality.py`, `perf_fix.py`.

> ⚠️ **Known fragility:** the editable source of truth and the shipped artifact are the *same* 1.8 MB file. A bad regex in any generator can corrupt the only copy of the data, with no clean rollback short of git. This build pattern is slated for a refactor that separates the editable data (`data/flows.json`) from the shipped `index.html` and replaces the in-place HTML surgery with structured JSON writes plus a provenance-validation step.

*(There is no `src/build.js` or `src/` pipeline — earlier documentation to that effect described a different, superseded project.)*

---

## License

- **Code** © the author, all rights reserved unless a `LICENSE` file states otherwise.
- **Datasets** remain under the terms of their respective providers (World Bank, UN agencies, FAO, USGS, IEA/EIA, OECD, TeleGeography, and the others listed above). Cited figures are used for visualization; consult each source for authoritative data and reuse terms.
