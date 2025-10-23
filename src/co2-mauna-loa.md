---
title: The Keeling Curve - Atmospheric CO2
toc: false
---

<style>
.hero {
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
}

.hero p {
  margin: 1rem 0 0;
  font-size: 1.2rem;
  color: var(--theme-foreground-muted);
}

* {
  overflow-anchor: none !important;
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

h2, h3 {
  max-width: 600px;
  margin-left: auto;
  margin-right: auto;
}

main > p, main > ul, main > ol {
  max-width: 600px;
  margin-left: auto;
  margin-right: auto;
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

## 📊 Atmospheric CO₂ Concentration Over Time

The famous Keeling Curve showing the relentless rise of atmospheric CO₂ from 1958 to present, measured at Mauna Loa Observatory. The zigzag pattern shows seasonal variations caused by Northern Hemisphere vegetation cycles.

```js
function co2Timeline({width} = {}) {
  return Plot.plot({
    width,
    height: 600,
    marginLeft: 70,
    marginRight: 30,
    marginTop: 40,
    marginBottom: 60,
    x: { label: "Year", grid: true },
    y: { label: "CO₂ Concentration (PPM)", grid: true, domain: [310, 440] },
    marks: [
      Plot.areaY(co2Data, { x: "year", y: "average", fill: "#7c3aed", fillOpacity: 0.15 }),
      Plot.line(co2Data, {
        x: "year",
        y: "average",
        stroke: "#7c3aed",
        strokeWidth: 1.5,
        tip: true,
        title: d => `${d.date.toLocaleDateString('en-US', { month: 'short', year: 'numeric' })}: ${d.average.toFixed(2)} PPM`
      }),
      Plot.line(co2Data.filter(d => d.trend !== null), { x: "year", y: "trend", stroke: "#dc2626", strokeWidth: 2.5 }),
      Plot.ruleY([350], { stroke: "#f59e0b", strokeWidth: 2, strokeDasharray: "4,4", opacity: 0.5 }),
      Plot.text([{x: 1985, y: 350}], { x: "x", y: "y", text: ["350 PPM"], fill: "#f59e0b", fontSize: 11, dy: -6 }),
      Plot.ruleY([400], { stroke: "#ef4444", strokeWidth: 2, strokeDasharray: "4,4", opacity: 0.5 }),
      Plot.text([{x: 2005, y: 400}], { x: "x", y: "y", text: ["400 PPM"], fill: "#ef4444", fontSize: 11, dy: -6 }),
      Plot.dot([latestReading], { x: "year", y: "average", r: 7, fill: "#dc2626", stroke: "white", strokeWidth: 2.5 })
    ]
  });
}
```

<div class="card">
  ${resize((width) => co2Timeline({width}))}
</div>

## 🔍 Understanding the Data

**The Purple Line (Measurements):** Shows actual monthly average CO₂ readings with visible seasonal oscillations. The zigzag pattern is caused by Northern Hemisphere vegetation absorbing CO₂ during summer growth and releasing it during winter decay.

**The Red Line (Trend):** Smoothed trend line that filters out seasonal variations to show the long-term upward trajectory of atmospheric CO₂ concentration.

**Key Milestones:**
- **Pre-industrial levels:** ~280 PPM
- **1958 (Start of measurements):** ${firstReading.average.toFixed(2)} PPM
- **350 PPM threshold:** Crossed in ${co2Data.find(d => d.average >= 350)?.date.getFullYear() || 'N/A'}
- **400 PPM threshold:** Crossed in ${co2Data.find(d => d.average >= 400)?.date.getFullYear() || 'N/A'}
- **Today:** ${latestReading.average.toFixed(2)} PPM - levels not seen in millions of years

---

**Data Source:** US Government's Earth System Research Laboratory, Global Monitoring Division https://datahub.io/core/co2-fossil-by-nation

**Location:** Mauna Loa Observatory, Hawaii (19.5°N, 155.6°W, 3397m elevation)

**Period:** March 1958 - Present

**About:** The Keeling Curve is named after scientist Charles David Keeling, who started these measurements in 1958. This dataset is one of the most important scientific records ever created, providing clear evidence of human impact on the atmosphere.
