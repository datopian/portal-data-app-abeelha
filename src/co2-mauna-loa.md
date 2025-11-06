---
title: The Keeling Curve - Atmospheric CO2
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

* {
  overflow-anchor: none !important;
}

.card, .card > *, .card svg, .card figure {
  overflow-anchor: none !important;
  contain: layout style paint;
}

svg text {
  fill: var(--theme-foreground) !important;
}

[id^="plot-tip-"],
[role="tooltip"],
[aria-live="polite"] {
  color: var(--theme-foreground) !important;
  background: var(--theme-background) !important;
}

[id^="plot-tip-"] *,
[role="tooltip"] *,
[aria-live="polite"] * {
  color: var(--theme-foreground) !important;
}

.card, .grid {
  max-width: 100% !important;
}
</style>

<div class="hero">
  <h1>🌍 The Keeling Curve</h1>
  <p>Atmospheric CO₂ measured at Mauna Loa Observatory, Hawaii (1958-2025)</p>
</div>

```js
const co2Raw = FileAttachment("data/co2-mm-mlo.csv").csv({typed: true});
const co2Data = (await co2Raw).map(d => ({
  date: new Date(d.Date),
  year: d["Decimal Date"],
  average: +d.Average,
  interpolated: +d.Interpolated,
  trend: +d.Trend > 0 ? +d.Trend : null
})).filter(d => !isNaN(d.average) && d.average > 0);

const latestReading = co2Data[co2Data.length - 1];
const firstReading = co2Data[0];
```

---

## Atmospheric CO₂ Concentration Over Time

The famous Keeling Curve showing the relentless rise of atmospheric CO₂ from 1958 to present, measured at Mauna Loa Observatory. The zigzag pattern shows seasonal variations caused by Northern Hemisphere vegetation cycles.

```js
const yearlyAverage = d3.rollups(
  co2Data,
  v => d3.mean(v, d => d.average),
  d => Math.floor(d.year)
).map(([year, avg]) => ({ year, average: avg }));
```

```js
function co2Timeline({width} = {}) {
  const height = 650;

  return Plot.plot({
    width,
    height,
    marginLeft: 70,
    marginRight: 40,
    marginTop: 50,
    marginBottom: 70,
    x: {
      label: "Year",
      grid: true,
      ticks: 10,
      tickFormat: "d"
    },
    y: {
      label: "CO₂ Concentration (parts per million)",
      grid: true,
      domain: [310, 435]
    },
    marks: [
      Plot.areaY(co2Data, {
        x: "year",
        y: "average",
        fill: "url(#co2Gradient)",
        fillOpacity: 0.3,
        curve: "catmull-rom"
      }),
      Plot.line(yearlyAverage, {
        x: "year",
        y: "average",
        stroke: "#3b82f6",
        strokeWidth: 3,
        strokeOpacity: 0.6,
        curve: "catmull-rom"
      }),
      Plot.line(co2Data, {
        x: "year",
        y: "average",
        stroke: "#0ea5e9",
        strokeWidth: 1.5,
        tip: true,
        curve: "catmull-rom",
        title: d => `${d.date.toLocaleDateString('en-US', { month: 'short', year: 'numeric' })}\n${d.average.toFixed(2)} PPM`
      }),
      Plot.ruleY([280], {
        stroke: "#22c55e",
        strokeWidth: 2,
        strokeDasharray: "6,4",
        opacity: 0.6
      }),
      Plot.text([{x: 1960, y: 280}], {
        x: "x",
        y: "y",
        text: ["← Pre-industrial (280 PPM)"],
        fill: "#22c55e",
        fontSize: 12,
        dy: -8,
        textAnchor: "start"
      }),
      Plot.ruleY([350], {
        stroke: "#f59e0b",
        strokeWidth: 2,
        strokeDasharray: "6,4",
        opacity: 0.6
      }),
      Plot.text([{x: 1990, y: 350}], {
        x: "x",
        y: "y",
        text: ["350 PPM (1988)"],
        fill: "#f59e0b",
        fontSize: 12,
        dy: -8
      }),
      Plot.ruleY([400], {
        stroke: "#ef4444",
        strokeWidth: 2,
        strokeDasharray: "6,4",
        opacity: 0.6
      }),
      Plot.text([{x: 2015, y: 400}], {
        x: "x",
        y: "y",
        text: ["400 PPM (2013)"],
        fill: "#ef4444",
        fontSize: 12,
        dy: -8
      }),
      Plot.dot([latestReading], {
        x: "year",
        y: "average",
        r: 8,
        fill: "#dc2626",
        stroke: "white",
        strokeWidth: 3
      }),
      Plot.text([latestReading], {
        x: "year",
        y: "average",
        text: d => `${d.average.toFixed(1)} PPM`,
        fill: "#dc2626",
        fontSize: 13,
        fontWeight: "bold",
        dx: 45,
        textAnchor: "start"
      }),
      Plot.ruleX([1958], {
        stroke: "#94a3b8",
        strokeWidth: 1.5,
        strokeDasharray: "3,3",
        opacity: 0.4
      }),
      Plot.text([{x: 1958, y: 432}], {
        x: "x",
        y: "y",
        text: ["Keeling starts\nmeasurements"],
        fill: "#64748b",
        fontSize: 11,
        textAnchor: "middle",
        lineAnchor: "bottom"
      })
    ]
  });
}
```

<svg width="0" height="0">
  <defs>
    <linearGradient id="co2Gradient" x1="0%" y1="0%" x2="0%" y2="100%">
      <stop offset="0%" style="stop-color:#0ea5e9;stop-opacity:0.9" />
      <stop offset="50%" style="stop-color:#06b6d4;stop-opacity:0.5" />
      <stop offset="100%" style="stop-color:#22c55e;stop-opacity:0.2" />
    </linearGradient>
  </defs>
</svg>

<div class="card" style="min-height: 700px;">
  ${resize((width) => co2Timeline({width}))}
</div>

## Understanding the Data

**Light Blue Line (Monthly Measurements):** Shows actual monthly average CO₂ readings with visible seasonal oscillations. The zigzag pattern is caused by Northern Hemisphere vegetation absorbing CO₂ during summer growth and releasing it during winter decay.

**Darker Blue Line (Yearly Average):** Smoothed annual trend line that filters out seasonal variations to show the long-term upward trajectory of atmospheric CO₂ concentration.

**Gradient Fill:** Visual representation showing the dramatic increase from near pre-industrial levels to today's unprecedented concentrations.

**Key Milestones:**
- **Pre-industrial levels (Green line):** ~280 PPM - the baseline for human civilization
- **1958 (Start of measurements):** ${firstReading.average.toFixed(2)} PPM - already 40 PPM above pre-industrial
- **350 PPM threshold (Orange line):** Crossed in 1988 - considered the safe upper limit by climate scientists
- **400 PPM threshold (Red line):** Crossed in 2013 - a symbolic and concerning milestone
- **Today:** ${latestReading.average.toFixed(2)} PPM - levels not seen in millions of years

**Rate of Change:** CO₂ is now increasing at approximately **2.5 PPM per year** - faster than any time in Earth's geological record.

---

**Data Source:** US Government's Earth System Research Laboratory, Global Monitoring Division https://datahub.io/core/co2-fossil-by-nation

**Location:** Mauna Loa Observatory, Hawaii (19.5°N, 155.6°W, 3397m elevation)

**Period:** March 1958 - Present

**About:** The Keeling Curve is named after scientist Charles David Keeling, who started these measurements in 1958. This dataset is one of the most important scientific records ever created, providing clear evidence of human impact on the atmosphere.
