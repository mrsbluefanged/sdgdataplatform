# SDG Data Platform (web)

A web version of the PSA x UNICEF Philippines SDG Data Platform Power BI prototype.
Home screen with all 17 goals, and a live interactive explorer for **Goal 1** driven
by the KoboToolbox form exports.

Everything is data-driven from `config.json` and the CSVs in `/data`. No build step,
no server-side code. Deploys as static files to GitHub Pages, Vercel, or SharePoint.

```
sdg-platform/
├── index.html          the whole app: charts, map, libraries, AND a built-in
│                       copy of the config + data (so it runs on its own)
├── config.json         <-- the file you normally edit to update the platform
├── assets/
│   ├── sdg-platform-logo.png   logo shown on the home page and subpages
│   └── sdg-goals/             SDG visual tiles used on the home page
└── data/               the KoboToolbox CSV exports (drop new ones here)
    ├── psa_phdsd.csv
    ├── psa_vsd.csv
    └── deped.csv
```

### One file, two ways to run
`index.html` is fully self-contained: the d3/topojson libraries, the map, the config,
the current data, and fallback copies of the logo and SDG goal tiles are all embedded.
So you can double-click it, email it, or drop it in a preview pane and it just works,
with no server.

When it *is* hosted alongside `config.json` and `/data` (GitHub Pages, SharePoint), it
prefers those external files. That is what keeps updating easy: replace a CSV on your
live site and the change shows up, without regenerating the HTML. The embedded copy is
just the offline fallback snapshot.


## Branding and visualization colors

The SDG Data Platform logo is stored at `/assets/sdg-platform-logo.png` and is shown on the home page and in the header of every goal page. To replace it, keep the same filename and path.

The home-page hero now uses one continuous white card behind the logo and descriptive text. The 17-goal grid uses SDG visual image tiles stored in `/assets/sdg-goals/goal-1.png` through `/assets/sdg-goals/goal-17.png`, with embedded fallback copies inside `index.html` for offline use.

Charts, maps, and ranked bars use the blue visualization palette instead of the SDG goal color. Red is reserved for warning/low-performance states such as the `Regressing` pace badge.

## How updating works

The app reads every form listed in `config.json`, splits each indicator to its goal by
the number prefix (`1.x` to Goal 1, `4.x` to Goal 4, and so on), and builds the whole
site from that. You do not touch `index.html` to add data.

### Update the data for a form
1. Export the form from KoboToolbox as CSV (the same 2-header-row layout as now).
2. Replace the file in `/data` (keep the same filename), or add the new filename and
   update the matching `file` path in `config.json`.
3. Commit / re-upload. Done.

### Add a new form (you have 47+)
Add one entry to the `forms` array in `config.json`:
```json
{ "id": "psa_labor", "file": "data/psa_labor.csv", "agency": "PSA - Labor Statistics" }
```
Any goals those indicators belong to light up automatically.

### Turn on another goal
In `config.json`, set a goal's `"status"` to `"live"`. It only becomes clickable once
there is actually data for it, so you can flip them on as forms arrive. Everything about
a goal (title, color, description) is editable there too.

### 2030 targets, units, and direction (the extra PBIX info)
The Power BI file carried 2030 targets and a pace-of-progress idea. Those live in the
`indicators` block of `config.json`, keyed by indicator code:
```json
"1.2.1": { "unit": "%", "direction": "down", "target2030": 11.7, "targetNote": "..." }
```
- `target2030` draws the dashed target line on the trend chart and drives the
  pace-of-progress badge (On track / Needs acceleration / Regressing).
- `direction` is `down` when lower is better (poverty) or `up` when higher is better
  (enrolment).
- Indicators with no entry still appear, just without a target line.

**Important:** the current `1.2.1` target is copied from the PBIX pace-of-progress table (`2030 Target = 11.7`).
Other indicators have no target line until they are added to `config.json`. Replace this with the official PSA SDG Watch / PDP target table before publishing.

## Run locally
Just open `index.html` in a browser. It runs on its own using the embedded data.

To test the live-update behavior (external files overriding the embedded copy), serve
the folder:
```
cd sdg-platform
python -m http.server 8000
# open http://localhost:8000
```

## Keeping the embedded snapshot in sync (optional)
The embedded fallback inside `index.html` is a point-in-time copy. When you update the
external `config.json` / CSVs on a hosted site, the live site is correct immediately and
you can ignore the embedded copy. Only refresh it if you also want the *standalone* file
(emailed or opened directly) to show the newest data. To do that, re-embed:
- the config into the `<script id="cfg-embedded">` block (plain JSON), and
- each CSV, base64-encoded, into its `<script id="data-FORMID">` block.
If you'd rather not touch the HTML, just distribute the folder instead of the lone file.

## Deploy to GitHub Pages
1. Put this folder in a repo (or a subfolder).
2. Settings → Pages → deploy from branch → root.
3. The live URL for Goal 1 is `.../index.html#/goal/1`.

## Notes on the data handling
- Regional values come from the `geo_disaggregation/<REGION>` columns. `PHI` is treated
  as the national series; the 17 regional columns feed the map, ranked bars, and table.
- ARMM rows are shown up to 2021 and BARMM from 2022, matching the form design. Both use
  the same base-map polygon.
- Negros Island Region appears in rankings/tables but has no polygon in the 2018
  PSA-NAMRIA ADM1 base map, so it is not shaded on the map (noted on the page).
- When a form has duplicate submissions for the same indicator/year/disaggregation, the
  most recently modified row wins (`_date_modified`).
- `direction`, target lines, and pace all recompute live from `config.json`.

## Swapping the base map
The ADM1 topology is embedded in `index.html` (a `<script id="topo">` block). It is the
same PSA-NAMRIA 2018 boundary set from the PBIX, simplified for web. If PSA updates
boundaries (e.g. NIR), regenerate a TopoJSON keyed by `ADM1_PCODE` / `ADM1_EN` and
replace that block.
