---
title: Global Sea Level Rise Dashboard
toc: false
---

<style>
.hero {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  padding: 3rem 2rem;
  background: var(--theme-background-alt);
  color: var(--theme-foreground);
  border-radius: 16px;
  margin-bottom: 2rem;
  border: 2px solid var(--theme-foreground-faint);
}

.hero h1 {
  margin: 0;
  font-size: 3rem;
  font-weight: 900;
  color: var(--theme-foreground-alt);
  max-width: 100%;
}

.hero p {
  margin: 1rem 0 0;
  font-size: 1.2rem;
  color: var(--theme-foreground-muted);
  max-width: 100%;
}

svg rect, svg path, svg circle, svg text {
  transition: opacity 0.5s ease-in-out, fill 0.5s ease-in-out;
}

* {
  overflow-anchor: none !important;
}

.card, .card > *, .card svg, .card figure {
  overflow-anchor: none !important;
  contain: layout style paint;
}

.card, .grid {
  max-width: 100% !important;
}

.card {
  min-height: 450px;
}

.dual-chart-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
  margin-bottom: 1rem;
}

@media (max-width: 768px) {
  .dual-chart-container {
    grid-template-columns: 1fr;
  }
}
</style>

<div class="hero">
  <h1>🌊 Global Mean Sea Level Rise Dashboard</h1>
  <p>This data contains cumulative changes in sea level for the world’s oceans since 1880, based on a combination of long-term tide gauge measurements and recent satellite measurements.</p>
</div>

```js
// Load data
const epaRaw = FileAttachment("data/epa-sea-level.csv").csv({typed: true});
const csiroRaw = FileAttachment("data/CSIRO_Recons_gmsl_yr_2019.csv").csv({typed: true});
const satRaw = FileAttachment("data/CSIRO_Alt_yearly.csv").csv({typed: true});
const glaciersRaw = FileAttachment("data/glaciers.csv").csv({typed: true});
```

```js
const epaData = (await epaRaw).map(d => ({
  year: +d.Year,
  value: +d["CSIRO Adjusted Sea Level"] * 25.4, // Convert inches to mm
  lower: +d["Lower Error Bound"] * 25.4,
  upper: +d["Upper Error Bound"] * 25.4,
  source: "EPA"
})).filter(d => d.year >= 1880 && d.year <= 2023 && !isNaN(d.value) && d.value !== 0);

const csiroData = (await csiroRaw).map(d => ({
  year: +d.Time,
  value: +d.GMSL,
  source: "CSIRO"
})).filter(d => d.year >= 1880);

const satData = (await satRaw).map(d => ({
  year: +d.Time,
  value: +d["GMSL (yearly)"],
  source: "Satellite"
})).filter(d => d.year >= 1993);

const glaciersData = (await glaciersRaw).map(d => ({
  year: +d.Year,
  value: +d["Mean cumulative mass balance"],
  observations: +d["Number of observations"]
})).filter(d => !isNaN(d.value) && d.year >= 1956);

const allData = [...epaData, ...csiroData, ...satData];
```

```js
// Get the latest year with actual data
const latestYear = Math.max(...csiroData.map(d => d.year));
```

```js
function Scrubber(values, {format = value => value, initial = 0, direction = 1, delay = null, autoplay = true, loop = true, loopDelay = null, alternate = false} = {}) {
  values = Array.from(values);
  const form = html`<form style="font: 12px var(--sans-serif); display: flex; height: 33px; align-items: center;"><button name=b type=button style="margin-right: 0.4em; width: 5em;"></button><label style="display: flex; align-items: center;"><input name=i type=range min=0 max=${values.length - 1} value=${initial} step=1 style="width: 180px;"><output name=o style="margin-left: 0.4em;"></output></label></form>`;
  let frame = null, timer = null, interval = null;
  const start = () => { form.b.textContent = "Pause"; if (delay === null) frame = requestAnimationFrame(tick); else interval = setInterval(tick, delay); };
  const stop = () => { form.b.textContent = "Play"; if (frame !== null) cancelAnimationFrame(frame), frame = null; if (timer !== null) clearTimeout(timer), timer = null; if (interval !== null) clearInterval(interval), interval = null; };
  const running = () => frame !== null || timer !== null || interval !== null;
  const step = () => { form.i.valueAsNumber = (form.i.valueAsNumber + direction + values.length) % values.length; form.i.dispatchEvent(new CustomEvent("input", {bubbles: true})); };
  const tick = () => { if (form.i.valueAsNumber === (direction > 0 ? values.length - 1 : direction < 0 ? 0 : NaN)) { if (!loop) return stop(); if (alternate) direction = -direction; if (loopDelay !== null) { if (frame !== null) cancelAnimationFrame(frame), frame = null; if (interval !== null) clearInterval(interval), interval = null; timer = setTimeout(() => (step(), start()), loopDelay); return; } } if (delay === null) frame = requestAnimationFrame(tick); step(); };
  form.i.oninput = event => { if (event && event.isTrusted && running()) stop(); form.value = values[form.i.valueAsNumber]; form.o.value = format(form.value, form.i.valueAsNumber, values); };
  form.b.onclick = () => { if (running()) return stop(); direction = alternate && form.i.valueAsNumber === values.length - 1 ? -1 : 1; form.i.valueAsNumber = (form.i.valueAsNumber + direction) % values.length; form.i.dispatchEvent(new CustomEvent("input", {bubbles: true})); start(); };
  form.i.oninput();
  if (autoplay) start(); else stop();
  Inputs.disposal(form).then(stop);
  return form;
}
```

```js
// Current stats
const currentEPA = epaData.find(d => d.year === selectedYear);
const currentCSIRO = csiroData.find(d => d.year === selectedYear);
const currentSat = satData.find(d => d.year === selectedYear);
```

---
## About This Data

**Global Average Absolute Sea Level Change, 1880-2014** from the US Environmental Protection Agency using data from CSIRO, 2015; NOAA, 2015.

This data contains cumulative changes in sea level for the world's oceans since 1880, based on a combination of long-term tide gauge measurements and recent satellite measurements.

It shows **average absolute sea level change**, which refers to the height of the ocean surface, regardless of whether nearby land is rising or falling.

Satellite data are based solely on measured sea level, while the long-term tide gauge data include a small correction factor because the size and shape of the oceans are changing slowly over time. *(On average, the ocean floor has been gradually sinking since the last Ice Age peak, 20,000 years ago.)*

---

## Data Sources

- **EPA**
  - Name: EPA (United States Environmental Protection Agency)
  - Web: http://www3.epa.gov/climatechange/images/indicator_downloads/sea-level_fig-1.csv

- **CSIRO**
  - Name: CSIRO (Commonwealth Scientific and Industrial Research Organization)
  - Web: http://www.cmar.csiro.au/sealevel/GMSL_SG_2011_up.html

- **Satellite**
  - Name: Satellite Altimetry Data
  - Web: CSIRO satellite altimetry measurements
---

## Sea Level Change Over Time

This line chart displays sea level measurements from all three datasets over time. **EPA** data (1880-2013) is shown in purple with uncertainty bands. **CSIRO** data (1880-2019) is shown in green. **Satellite** data (1993-2020) is shown in red. All data is displayed in millimeters (EPA converted from inches).

```js
const selectedDataset = view(Inputs.select(
  ["All", "EPA", "CSIRO", "Satellite"],
  {label: "Source", value: "All"}
));
```

```js
// Filter data
const filteredData = allData.filter(d => {
  if (selectedDataset === "All") return d.year <= selectedYear;
  if (selectedDataset === "EPA") return d.source === "EPA" && d.year <= selectedYear;
  if (selectedDataset === "CSIRO") return d.source === "CSIRO" && d.year <= selectedYear;
  if (selectedDataset === "Satellite") return d.source === "Satellite" && d.year <= selectedYear;
  return false;
});
```

```js
function seaLevelChart(data, {width} = {}) {
  return Plot.plot({
    title: "Global Mean Sea Level Rise",
    width,
    height: 400,
    marginLeft: 60,
    x: {label: "Year", grid: true, domain: [1880, latestYear]},
    y: {label: "Sea Level Change (mm)", grid: true},
    color: {
      legend: true,
      domain: ["EPA", "CSIRO", "Satellite"],
      range: ["#4f46e5", "#059669", "#dc2626"]
    },
    marks: [
      // Uncertainty band for EPA
      Plot.areaY(
        data.filter(d => d.source === "EPA" && d.lower !== undefined),
        {x: "year", y1: "lower", y2: "upper", fill: "#e0e7ff", fillOpacity: 0.3}
      ),
      // Lines for each dataset
      Plot.line(data, {
        x: "year",
        y: "value",
        stroke: "source",
        strokeWidth: 2,
        curve: "catmull-rom"
      }),
      // Current year marker
      Plot.ruleX([selectedYear], {stroke: "#f59e0b", strokeWidth: 2}),
      // Interactive dots
      Plot.dot(data, Plot.pointerX({
        x: "year",
        y: "value",
        stroke: "source",
        r: 4,
        title: d => `${d.source} (${d.year}): ${d.value.toFixed(1)} mm`
      })),
      Plot.ruleY([0])
    ]
  });
}
```

<div class="card">
  ${resize((width) => seaLevelChart(filteredData, {width}))}
</div>

```js
const years = d3.range(1880, latestYear + 1);
const selectedYear = view(Scrubber(years, {
  delay: 150,
  loop: false,
  initial: years.length - 1,
  autoplay: false,
  format: d => d
}));
```

---

## ❄️ Sea Level Rise & Glacier Mass Balance Correlation

This synchronized visualization shows the relationship between global sea level rise and glacier mass loss from 1956 to 2019. As glaciers lose mass (shown as increasingly negative values), sea level rises. The data reveals a clear correlation: glacier melt contributes significantly to sea level rise. Values are in millimeters for sea level and meters of water equivalent for glacier mass balance.

```js
const correlationYears = d3.range(1956, 2020);
const selectedCorrelationYear = view(Scrubber(correlationYears, {
  delay: 150,
  loop: false,
  initial: correlationYears.length - 1,
  autoplay: false,
  format: d => d
}));
```

```js
const correlationSeaLevel = csiroData.filter(d => d.year >= 1956 && d.year <= 2019 && d.year <= selectedCorrelationYear);
const correlationGlaciers = glaciersData.filter(d => d.year >= 1956 && d.year <= 2019 && d.year <= selectedCorrelationYear);
```

```js
function seaLevelCorrelationChart({width} = {}) {
  return Plot.plot({
    title: "Sea Level Rise (CSIRO)",
    width,
    height: 350,
    marginLeft: 60,
    marginRight: 20,
    x: {label: "Year", grid: true, domain: [1956, 2019]},
    y: {label: "Sea Level Change (mm)", grid: true},
    marks: [
      Plot.line(correlationSeaLevel, {
        x: "year",
        y: "value",
        stroke: "#059669",
        strokeWidth: 3,
        curve: "catmull-rom"
      }),
      Plot.dot(correlationSeaLevel, {
        x: "year",
        y: "value",
        fill: "#059669",
        r: 2
      }),
      Plot.ruleX([selectedCorrelationYear], {stroke: "#f59e0b", strokeWidth: 2, strokeDasharray: "4,4"}),
      Plot.dot(
        correlationSeaLevel.filter(d => d.year === selectedCorrelationYear),
        {
          x: "year",
          y: "value",
          fill: "#f59e0b",
          r: 6,
          stroke: "white",
          strokeWidth: 2
        }
      ),
      Plot.ruleY([0], {stroke: "#94a3b8", strokeDasharray: "2,2"})
    ]
  });
}
```

```js
function glacierCorrelationChart({width} = {}) {
  return Plot.plot({
    title: "Glacier Mass Balance (WGMS)",
    width,
    height: 350,
    marginLeft: 60,
    marginRight: 20,
    x: {label: "Year", grid: true, domain: [1956, 2019]},
    y: {label: "Cumulative Mass Balance (m)", grid: true},
    marks: [
      Plot.line(correlationGlaciers, {
        x: "year",
        y: "value",
        stroke: "#3b82f6",
        strokeWidth: 3,
        curve: "catmull-rom"
      }),
      Plot.dot(correlationGlaciers, {
        x: "year",
        y: "value",
        fill: "#3b82f6",
        r: 2
      }),
      Plot.ruleX([selectedCorrelationYear], {stroke: "#f59e0b", strokeWidth: 2, strokeDasharray: "4,4"}),
      Plot.dot(
        correlationGlaciers.filter(d => d.year === selectedCorrelationYear),
        {
          x: "year",
          y: "value",
          fill: "#f59e0b",
          r: 6,
          stroke: "white",
          strokeWidth: 2
        }
      ),
      Plot.ruleY([0], {stroke: "#94a3b8", strokeDasharray: "2,2"})
    ]
  });
}
```

<div class="dual-chart-container">
  <div class="card">
    ${resize((width) => seaLevelCorrelationChart({width}))}
  </div>
  <div class="card">
    ${resize((width) => glacierCorrelationChart({width}))}
  </div>
</div>

---

## 🌊 Horizon View - Compact Multi-Dataset Comparison

This horizon chart shows all three datasets in a compact, layered format. Each row represents one dataset (EPA, CSIRO, Satellite), with blue bands indicating the magnitude of sea level rise. Darker/higher bands represent higher sea level values. All data is displayed in millimeters (EPA converted from inches).

```js
function horizonChart({width} = {}) {
  const bands = 4;
  const bandHeight = 50;
  const height = bandHeight * bands;
  const datasets = ["EPA", "CSIRO", "Satellite"];

  // Group data by source
  const epaOnly = epaData.filter(d => d.value > 0);
  const csiroOnly = csiroData.filter(d => d.value > 0);
  const satOnly = satData.filter(d => d.value > 0);

  const dataBySource = {
    "EPA": epaOnly,
    "CSIRO": csiroOnly,
    "Satellite": satOnly
  };

  // Calculate max value for scaling
  const maxValue = d3.max([...epaOnly, ...csiroOnly, ...satOnly], d => d.value);

  return Plot.plot({
    width,
    height: height * 3 + 120,
    marginLeft: 100,
    marginRight: 20,
    marginTop: 30,
    marginBottom: 30,
    fy: {
      domain: datasets,
      label: null
    },
    x: {
      label: "Year",
      domain: [1880, latestYear],
      grid: true
    },
    y: {
      axis: null,
      domain: [0, bandHeight]
    },
    color: {
      scheme: "Blues",
      domain: [0, bands]
    },
    facet: {
      marginLeft: 100
    },
    marks: [
      // Draw overlapping bands for each dataset
      ...datasets.flatMap(source => {
        const data = dataBySource[source];
        return Array.from({length: bands}, (_, bandIndex) => {
          const threshold = (maxValue / bands) * bandIndex;

          return Plot.areaY(data, {
            fy: () => source,
            x: "year",
            y1: 0,
            y2: d => {
              const excess = d.value - threshold;
              return excess > 0 ? Math.min(bandHeight, (excess / (maxValue / bands)) * bandHeight) : 0;
            },
            fill: d3.interpolateBlues((bandIndex + 1) / bands),
            fillOpacity: 0.8,
            curve: "catmull-rom"
          });
        });
      }),
      // Source labels
      Plot.text(datasets, (d, i) => [{
        fy: d,
        x: 1880,
        y: bandHeight / 2,
        text: d,
        source: d
      }], {
        x: "x",
        y: "y",
        text: "text",
        fy: "fy",
        dx: -10,
        textAnchor: "end",
        fill: d => d === "EPA" ? "#4f46e5" : d === "CSIRO" ? "#059669" : "#dc2626",
        fontSize: 12,
        fontWeight: "bold"
      }),
      // Interactive tooltip
      Plot.tip(
        datasets.flatMap(source =>
          dataBySource[source].map(d => ({...d, source}))
        ),
        Plot.pointerX({
          fy: "source",
          x: "year",
          y: () => bandHeight / 2,
          title: d => `${d.source}\n${d.year}: ${d.value.toFixed(1)} mm`,
          fill: "white",
          stroke: "#64748b"
        })
      )
    ]
  });
}
```

<div class="card">
  ${resize((width) => horizonChart({width}))}
</div>

---

## 🔄 Connected Scatterplot - Sea Level vs Rate of Change

This visualization shows the annual rate of change in sea level using data from the **CSIRO Reconstructed GMSL** dataset. The rate of change is calculated by subtracting each year's sea level value from the previous year's value, showing the year-over-year difference from the data.

```js
// Prepare scatterplot data with rate of change
const scatterData = csiroData.slice(1).map((d, i) => {
  const prevValue = csiroData[i].value;
  const rate = d.value - prevValue;
  const decade = Math.floor(d.year / 10) * 10;

  return {
    year: d.year,
    value: d.value,
    rate: rate,
    decade: decade,
    decadeLabel: `${decade}s`
  };
}).filter(d => d.year >= 1880 && d.year <= latestYear);
```

```js
function connectedScatterplot({width} = {}) {
  const height = Math.min(600, width * 0.7);

  return Plot.plot({
    width,
    height,
    marginLeft: 60,
    marginRight: 60,
    marginTop: 40,
    marginBottom: 60,
    grid: true,
    x: {
      label: "Year",
      domain: [1880, latestYear],
      nice: true
    },
    y: {
      label: "Annual Rate of Change (mm/year)",
      nice: true
    },
    marks: [
      // Connected path showing temporal progression
      Plot.line(scatterData, {
        x: "year",
        y: "rate",
        stroke: "#059669",
        strokeWidth: 2,
        curve: "catmull-rom"
      }),
      // Dots for each year
      Plot.dot(scatterData, {
        x: "year",
        y: "rate",
        fill: "#059669",
        r: 3,
        stroke: "white",
        strokeWidth: 1,
        tip: true,
        title: d => `${d.year}\nSea Level: ${d.value.toFixed(1)} mm\nRate: ${d.rate >= 0 ? '+' : ''}${d.rate.toFixed(1)} mm/year`
      }),
      // Reference lines
      Plot.ruleY([0], {stroke: "#94a3b8", strokeDasharray: "4,4"})
    ]
  });
}
```

<div class="card">
  ${resize((width) => connectedScatterplot({width}))}
</div>

---

## 📅 Calendar Heatmap - Annual Rate of Change Patterns

This heatmap shows the annual rate of change using **CSIRO Reconstructed GMSL** data. Each cell represents one year, organized by decades. The color indicates the rate of change (blue = decrease, red = increase) calculated by subtracting consecutive year values. Data is in millimeters.

```js
// Prepare calendar data with rate of change
const calendarData = csiroData.slice(1).map((d, i) => {
  const prevValue = csiroData[i].value;
  const rate = d.value - prevValue;
  const decade = Math.floor(d.year / 10) * 10;
  const yearInDecade = d.year % 10;

  return {
    year: d.year,
    rate: rate,
    decade: decade,
    yearInDecade: yearInDecade,
    decadeLabel: `${decade}s`,
    absValue: d.value
  };
}).filter(d => d.year >= 1880 && d.year <= latestYear);
```

```js
function calendarHeatmap({width} = {}) {
  const cellSize = Math.min(40, (width - 120) / 14);
  const decades = Array.from(new Set(calendarData.map(d => d.decade))).sort();

  return Plot.plot({
    width,
    height: decades.length * (cellSize + 4) + 60,
    marginLeft: 80,
    marginRight: 40,
    marginTop: 30,
    marginBottom: 30,
    padding: 0,
    x: {
      label: null,
      domain: d3.range(0, 10),
      axis: "top",
      tickFormat: d => d
    },
    fy: {
      label: null,
      domain: decades.map(d => `${d}s`),
      tickFormat: d => d
    },
    color: {
      scheme: "RdBu",
      reverse: true,
      domain: d3.extent(calendarData, d => d.rate),
      legend: true,
      label: "Annual Change (mm)"
    },
    marks: [
      // Heatmap cells
      Plot.cell(calendarData, {
        x: "yearInDecade",
        fy: "decadeLabel",
        fill: "rate",
        inset: 2,
        rx: 4,
        tip: true,
        title: d => `${d.year}\nRate: ${d.rate >= 0 ? '+' : ''}${d.rate.toFixed(1)} mm/year\nTotal: ${d.absValue.toFixed(1)} mm`
      }),
      // Year labels for first and last of each decade
      Plot.text(calendarData.filter(d => d.yearInDecade === 0 || d.yearInDecade === 9), {
        x: "yearInDecade",
        fy: "decadeLabel",
        text: "year",
        fontSize: 9,
        fill: "#1f2937",
        fontWeight: "bold",
        textAnchor: "middle",
        dy: 1
      })
    ]
  });
}
```

<div class="card">
  ${resize((width) => calendarHeatmap({width}))}
</div>

---

## Decade Comparison

This bar chart shows the average sea level for each decade. Data uses **EPA** for decades 1880-2009 and **CSIRO** for 2010-2019. The average is calculated by summing all year values within each decade and dividing by 10. All values are in millimeters.

```js
const decadeData = [
  {decade: "1880-1889", value: epaData.filter(d => d.year >= 1880 && d.year < 1890).reduce((sum, d) => sum + d.value, 0) / 10},
  {decade: "1890-1899", value: epaData.filter(d => d.year >= 1890 && d.year < 1900).reduce((sum, d) => sum + d.value, 0) / 10},
  {decade: "1900-1909", value: epaData.filter(d => d.year >= 1900 && d.year < 1910).reduce((sum, d) => sum + d.value, 0) / 10},
  {decade: "1910-1919", value: epaData.filter(d => d.year >= 1910 && d.year < 1920).reduce((sum, d) => sum + d.value, 0) / 10},
  {decade: "1920-1929", value: epaData.filter(d => d.year >= 1920 && d.year < 1930).reduce((sum, d) => sum + d.value, 0) / 10},
  {decade: "1930-1939", value: epaData.filter(d => d.year >= 1930 && d.year < 1940).reduce((sum, d) => sum + d.value, 0) / 10},
  {decade: "1940-1949", value: epaData.filter(d => d.year >= 1940 && d.year < 1950).reduce((sum, d) => sum + d.value, 0) / 10},
  {decade: "1950-1959", value: epaData.filter(d => d.year >= 1950 && d.year < 1960).reduce((sum, d) => sum + d.value, 0) / 10},
  {decade: "1960-1969", value: epaData.filter(d => d.year >= 1960 && d.year < 1970).reduce((sum, d) => sum + d.value, 0) / 10},
  {decade: "1970-1979", value: epaData.filter(d => d.year >= 1970 && d.year < 1980).reduce((sum, d) => sum + d.value, 0) / 10},
  {decade: "1980-1989", value: epaData.filter(d => d.year >= 1980 && d.year < 1990).reduce((sum, d) => sum + d.value, 0) / 10},
  {decade: "1990-1999", value: epaData.filter(d => d.year >= 1990 && d.year < 2000).reduce((sum, d) => sum + d.value, 0) / 10},
  {decade: "2000-2009", value: epaData.filter(d => d.year >= 2000 && d.year < 2010).reduce((sum, d) => sum + d.value, 0) / 10},
  {decade: "2010-2019", value: csiroData.filter(d => d.year >= 2010 && d.year < 2020).reduce((sum, d) => sum + d.value, 0) / 10},
];
```

```js
function decadeChart(data, {width} = {}) {
  return Plot.plot({
    title: "Average Sea Level by Decade",
    width,
    height: 350,
    marginLeft: 60,
    marginBottom: 80,
    x: {label: null, tickRotate: -45},
    y: {label: "Average Sea Level (mm)", grid: true},
    marks: [
      Plot.barY(data, {
        x: "decade",
        y: "value",
        fill: d => d.value < 25 ? "#059669" : d.value < 75 ? "#f59e0b" : "#dc2626",
        tip: true
      }),
      Plot.ruleY([0])
    ]
  });
}
```

<div class="card">
  ${resize((width) => decadeChart(decadeData, {width}))}
</div>

---

## Key Findings

We estimate the rise in global average sea level from satellite altimeter data for **1993–2009** and from coastal and island sea-level measurements from **1880 to 2009**.

For 1993–2009 and after correcting for glacial isostatic adjustment, the estimated rate of rise is **3.2 ± 0.4 mm year⁻¹** from the satellite data and **2.8 ± 0.8 mm year⁻¹** from the in situ data.

The global average sea-level rise from 1880 to 2009 is about **210 mm**. The linear trend from 1900 to 2009 is **1.7 ± 0.2 mm year⁻¹** and since 1961 is **1.9 ± 0.4 mm year⁻¹**.

There is considerable variability in the rate of rise during the twentieth century but there has been a **statistically significant acceleration** since 1880 and 1900 of **0.009 ± 0.003 mm year⁻²** and **0.009 ± 0.004 mm year⁻²**, respectively.

Since the start of the altimeter record in 1993, global average sea level rose at a rate near the **upper end of the sea level projections** of the Intergovernmental Panel on Climate Change's Third and Fourth Assessment Reports. However, the reconstruction indicates there was **little net change in sea level from 1990 to 1993**, most likely as a result of the volcanic eruption of **Mount Pinatubo in 1991**.

## References

- https://datahub.io/core/sea-level-rise
- https://github.com/datasets/sea-level-rise
- https://www.epa.gov/climate-indicators/climate-change-indicators-sea-level
- https://www.cmar.csiro.au/sealevel/sl_hist_last_decades.html

---

**Note:** This shows global mean sea level. Local changes vary due to land subsidence, ocean currents, and regional climate patterns.
