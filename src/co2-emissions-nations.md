---
title: Global CO2 Emissions by Nation
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

.card, .grid {
  max-width: 100% !important;
}

.card {
  min-height: 600px;
}
</style>

<div class="hero">
  <h1>🌍 Global CO₂ Emissions</h1>
  <p>Fossil Fuel & Cement Emissions by Nation (1751-2020)</p>
</div>

```js
// Load emissions data
const emissionsRaw = FileAttachment("data/fossil-fuel-co2-emissions-by-nation.csv").csv({typed: true});
const emissionsData = (await emissionsRaw).map(d => ({
  year: +d.Year,
  country: d.Country,
  total: +d.Total || 0,
  solidFuel: +d["Solid Fuel"] || 0,
  liquidFuel: +d["Liquid Fuel"] || 0,
  gasFuel: +d["Gas Fuel"] || 0,
  cement: +d.Cement || 0,
  gasFlaring: +d["Gas Flaring"] || 0,
  perCapita: +d["Per Capita"] || 0
})).filter(d => d.total > 0);

// Get latest year data
const latestYear = d3.max(emissionsData, d => d.year);
const latestYearData = emissionsData.filter(d => d.year === latestYear);

// Calculate global total
const globalTotal = d3.sum(latestYearData, d => d.total);

// Get top emitters
const topEmitters = latestYearData
  .sort((a, b) => b.total - a.total)
  .slice(0, 20);

// Get top per capita
const topPerCapita = latestYearData
  .filter(d => d.perCapita > 0)
  .sort((a, b) => b.perCapita - a.perCapita)
  .slice(0, 10);
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

---

## 📊 Emissions Breakdown by Source

Explore how different fuel types contribute to each country's total emissions for the selected year: **${selectedYearBreakdown}**.

```js
const yearDataBreakdown = emissionsData
  .filter(d => d.year === selectedYearBreakdown && d.total > 0)
  .sort((a, b) => b.total - a.total)
  .slice(0, 20);
```

```js
const topCountriesForBreakdown = yearDataBreakdown.slice(0, 12);

function emissionsBreakdown({width} = {}) {
  const breakdownData = topCountriesForBreakdown.flatMap(d => [
    { country: d.country, type: "Solid Fuel", value: d.solidFuel },
    { country: d.country, type: "Liquid Fuel", value: d.liquidFuel },
    { country: d.country, type: "Gas Fuel", value: d.gasFuel },
    { country: d.country, type: "Cement", value: d.cement },
    { country: d.country, type: "Gas Flaring", value: d.gasFlaring }
  ]);

  return Plot.plot({
    width,
    height: 550,
    marginLeft: 150,
    marginBottom: 80,
    marginTop: 20,
    x: {
      label: "Emissions (Million Metric Tons C)",
      grid: true
    },
    y: {
      label: null
    },
    color: {
      legend: true,
      domain: ["Solid Fuel", "Liquid Fuel", "Gas Fuel", "Cement", "Gas Flaring"],
      range: ["#64748b", "#f59e0b", "#ef4444", "#8b5cf6", "#ec4899"],
      label: null
    },
    marks: [
      Plot.barX(breakdownData, {
        x: "value",
        y: "country",
        fill: "type",
        tip: true,
        title: d => `${d.country} - ${d.type}\n${d.value.toLocaleString()} million metric tons`,
        sort: { y: "x", reverse: true }
      }),
      Plot.ruleX([0])
    ]
  });
}
```

<div class="card">
  ${resize((width) => emissionsBreakdown({width}))}
</div>

```js
const yearsBreakdown = d3.range(1751, latestYear + 1);
const selectedYearBreakdown = view(Scrubber(yearsBreakdown, {
  delay: 150,
  loop: false,
  initial: yearsBreakdown.length - 1,
  autoplay: false,
  format: d => d
}));
```

---

## 🌐 Global Emissions Treemap

A treemap showing the relative size of emissions by country for year **${selectedYearTreemap}**. Larger blocks represent higher emissions. Hover to see detailed information.

```js
const yearDataTreemap = emissionsData
  .filter(d => d.year === selectedYearTreemap && d.total > 0)
  .sort((a, b) => b.total - a.total)
  .slice(0, 20);
```

```js
function emissionsTreemap({width} = {}) {
  const height = 650;

  // Prepare hierarchical data - show top 30 countries
  const topCountriesTreemap = yearDataTreemap.slice(0, 30);
  const hierarchyData = {
    name: "Global",
    children: topCountriesTreemap.map(d => ({
      name: d.country,
      value: d.total,
      perCapita: d.perCapita
    }))
  };

  const root = d3.hierarchy(hierarchyData)
    .sum(d => d.value)
    .sort((a, b) => b.value - a.value);

  d3.treemap()
    .size([width, height])
    .paddingInner(3)
    .paddingOuter(6)
    .round(true)(root);

  const colorScale = d3.scaleSequential(d3.interpolateRdYlGn)
    .domain([d3.max(topCountriesTreemap, d => d.total), 0]);

  const svg = d3.create("svg")
    .attr("width", width)
    .attr("height", height)
    .attr("viewBox", [0, 0, width, height])
    .style("background", "#ffffff")
    .style("font", "13px sans-serif");

  const leaf = svg.selectAll("g")
    .data(root.leaves())
    .join("g")
      .attr("transform", d => `translate(${d.x0},${d.y0})`);

  leaf.append("rect")
    .attr("fill", d => colorScale(d.data.value))
    .attr("fill-opacity", 0.85)
    .attr("width", d => d.x1 - d.x0)
    .attr("height", d => d.y1 - d.y0)
    .attr("stroke", "#e5e7eb")
    .attr("stroke-width", 3)
    .attr("rx", 4)
    .style("cursor", "pointer")
    .on("mouseenter", function() {
      d3.select(this)
        .attr("fill-opacity", 1)
        .attr("stroke", "#fbbf24")
        .attr("stroke-width", 4);
    })
    .on("mouseleave", function() {
      d3.select(this)
        .attr("fill-opacity", 0.85)
        .attr("stroke", "#e5e7eb")
        .attr("stroke-width", 3);
    })
    .append("title")
    .text(d => `${d.data.name}\n${d.data.value.toLocaleString()} million metric tons total\n${d.data.perCapita > 0 ? d.data.perCapita.toFixed(2) + ' tons per capita' : 'N/A per capita'}`);

  // Country name
  leaf.append("text")
    .attr("x", 8)
    .attr("y", 22)
    .attr("fill", "black")
    .attr("font-weight", "bold")
    .attr("font-size", d => {
      const width = d.x1 - d.x0;
      const height = d.y1 - d.y0;
      return Math.min(width / 8, height / 3.5, 16);
    })
    .text(d => d.data.name)
    .each(function(d) {
      const width = d.x1 - d.x0;
      const textLength = this.getComputedTextLength();
      if (textLength > width - 16) {
        const maxLength = Math.floor((width - 16) / (textLength / d.data.name.length));
        d3.select(this).text(d.data.name.substring(0, maxLength) + "...");
      }
    });

  // Emissions value
  leaf.append("text")
    .attr("x", 8)
    .attr("y", 42)
    .attr("fill", "black")
    .attr("font-size", d => {
      const width = d.x1 - d.x0;
      const height = d.y1 - d.y0;
      return Math.min(width / 10, height / 5, 14);
    })
    .attr("font-weight", "600")
    .attr("opacity", 0.85)
    .text(d => {
      const val = d.data.value;
      if (val >= 1000) return `${(val / 1000).toFixed(1)}K MT`;
      return `${val.toFixed(0)} MT`;
    });

  return svg.node();
}
```

<div class="card">
  ${resize((width) => emissionsTreemap({width}))}
</div>

```js
const yearsTreemap = d3.range(1751, latestYear + 1);
const selectedYearTreemap = view(Scrubber(yearsTreemap, {
  delay: 150,
  loop: false,
  initial: yearsTreemap.length - 1,
  autoplay: false,
  format: d => d
}));
```

---

## 📈 Historical Trends: Top 5 Emitters Over Time

Watch how the biggest emitters have evolved from 1850 to 2020. This log-scale chart shows the dramatic rise of emissions during the industrial revolution and modern era.

```js
// Get top 5 countries from latest year
const top5Countries = topEmitters.slice(0, 5).map(d => d.country);

// Get their historical data from 1850
const historicalData = emissionsData.filter(d =>
  top5Countries.includes(d.country) &&
  d.year >= 1850
);

function historicalTrends({width} = {}) {
  return Plot.plot({
    width,
    height: 600,
    marginLeft: 80,
    marginRight: 20,
    marginTop: 30,
    marginBottom: 70,
    x: {
      label: "Year",
      grid: true,
      tickFormat: "d"
    },
    y: {
      label: "Total Emissions (Million Metric Tons C)",
      grid: true,
      type: "log",
      ticks: [1, 10, 100, 1000, 10000]
    },
    color: {
      legend: true,
      domain: top5Countries,
      range: ["#ef4444", "#f59e0b", "#22c55e", "#3b82f6", "#8b5cf6"],
      label: null
    },
    marks: [
      // Area fills under each line
      Plot.areaY(historicalData, {
        x: "year",
        y: "total",
        z: "country",
        fill: "country",
        fillOpacity: 0.15,
        curve: "catmull-rom"
      }),
      // Main lines
      Plot.line(historicalData, {
        x: "year",
        y: "total",
        stroke: "country",
        strokeWidth: 3.5,
        curve: "catmull-rom",
        tip: true,
        title: d => `${d.country} (${d.year})\n${d.total.toLocaleString()} million metric tons`
      }),
      // Add dots for latest year
      Plot.dot(
        historicalData.filter(d => d.year === latestYear),
        {
          x: "year",
          y: "total",
          fill: "country",
          r: 6,
          stroke: "white",
          strokeWidth: 2
        }
      ),
      // Historical reference lines
      Plot.ruleX([1900], {
        stroke: "#64748b",
        strokeWidth: 1,
        strokeDasharray: "4,4",
        opacity: 0.5
      }),
      Plot.ruleX([1950], {
        stroke: "#64748b",
        strokeWidth: 1,
        strokeDasharray: "4,4",
        opacity: 0.5
      }),
      Plot.ruleX([2000], {
        stroke: "#64748b",
        strokeWidth: 1,
        strokeDasharray: "4,4",
        opacity: 0.5
      })
    ]
  });
}
```

<div class="card">
  ${resize((width) => historicalTrends({width}))}
</div>

---

**Data Source:** Carbon Dioxide Information Analysis Center (CDIAC) at Appalachian State University
**Citation:** Hefner, M., and G. Marland. 2023. Global, Regional, and National Fossil-Fuel CO2 Emissions: 1751-2020.
**Period:** 1751-2020
