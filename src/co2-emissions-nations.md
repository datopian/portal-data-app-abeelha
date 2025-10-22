---
title: Global CO2 Emissions by Nation
toc: false
---

<style>
.hero {
  text-align: center;
  padding: 3rem 2rem;
  background: linear-gradient(135deg, #dc2626 0%, #ea580c 50%, #f59e0b 100%);
  color: white;
  border-radius: 16px;
  margin-bottom: 2rem;
  box-shadow: 0 10px 40px rgba(220, 38, 38, 0.4);
}

.hero h1 {
  margin: 0;
  font-size: 3rem;
  font-weight: 900;
  color: white;
  text-shadow: 2px 2px 8px rgba(0,0,0,0.4);
}

.hero p {
  margin: 1rem 0 0;
  font-size: 1.2rem;
  opacity: 0.95;
}

.stat-card {
  background: linear-gradient(135deg, #0f172a 0%, #1e293b 100%);
  padding: 2rem;
  border-radius: 12px;
  color: white;
  text-align: center;
  box-shadow: 0 4px 20px rgba(0,0,0,0.3);
  border: 2px solid #334155;
}

.stat-value {
  font-size: 3rem;
  font-weight: 900;
  line-height: 1;
  margin: 0;
  background: linear-gradient(135deg, #f59e0b 0%, #ef4444 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.stat-label {
  font-size: 1rem;
  opacity: 0.8;
  margin-top: 0.5rem;
}

.country-highlight {
  background: linear-gradient(135deg, #7c3aed 0%, #c026d3 100%);
  padding: 1.5rem;
  border-radius: 12px;
  color: white;
  margin: 2rem 0;
  text-align: center;
}

* {
  overflow-anchor: none !important;
}

.card, .card > *, .card svg, .card figure {
  overflow-anchor: none !important;
  contain: layout style paint;
}

.year-control-overlay {
  position: fixed;
  right: 20px;
  top: 50%;
  transform: translateY(-50%);
  background: linear-gradient(135deg, #dc2626 0%, #ea580c 100%);
  padding: 1rem 1rem;
  border-radius: 12px;
  box-shadow: -4px 0 20px rgba(0, 0, 0, 0.3);
  z-index: 1000;
  width: 160px;
  backdrop-filter: blur(10px);
}

.year-control-overlay h3 {
  color: white;
  margin: 0 0 0.5rem 0;
  font-size: 0.9rem;
  text-align: center;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.2);
}

.year-control-overlay .year-display-overlay {
  font-size: 2rem;
  font-weight: bold;
  color: white;
  text-align: center;
  margin: 0.75rem 0 0.25rem 0;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
  letter-spacing: 1px;
}

.year-control-overlay input[type="range"] {
  width: 100%;
  margin: 0.5rem 0;
  cursor: pointer;
  accent-color: #fbbf24;
}

.year-control-overlay input[type="number"] {
  background: rgba(255, 255, 255, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.3);
  color: white !important;
  padding: 0.3rem;
  border-radius: 4px;
  text-align: center;
  font-size: 0.9rem;
  width: 100%;
}

.year-control-overlay label,
.year-control-overlay span,
.year-control-overlay div {
  color: white !important;
}

.year-control-overlay form {
  width: 100%;
}

.year-control-overlay output {
  color: white !important;
}

/* Simple text colors */
svg text {
  fill: black !important;
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

<div class="year-control-overlay">
  <h3>📅 Year Control</h3>

```js
const selectedYear = view(Inputs.range([1751, latestYear], {
  step: 1,
  value: latestYear
}));
```

  <div class="year-display-overlay">${selectedYear}</div>
</div>

```js
// Preserve scroll position during year changes
{
  let preservedScrollY = window.scrollY || window.pageYOffset;
  let isUserScrolling = false;
  let scrollTimeout;

  window.addEventListener('scroll', () => {
    if (!isUserScrolling) {
      isUserScrolling = true;
      preservedScrollY = window.scrollY || window.pageYOffset;
    }
    clearTimeout(scrollTimeout);
    scrollTimeout = setTimeout(() => {
      isUserScrolling = false;
      preservedScrollY = window.scrollY || window.pageYOffset;
    }, 100);
  }, { passive: true });

  const observer = new MutationObserver(() => {
    const currentScroll = window.scrollY || window.pageYOffset;
    if (!isUserScrolling && Math.abs(currentScroll - preservedScrollY) > 5) {
      window.scrollTo(0, preservedScrollY);
    }
  });

  observer.observe(document.body, {
    childList: true,
    subtree: true,
    attributes: false
  });
}
```

```js
const yearData = emissionsData
  .filter(d => d.year === selectedYear && d.total > 0)
  .sort((a, b) => b.total - a.total)
  .slice(0, 20);
```

---

## 📊 Emissions Breakdown by Source

Explore how different fuel types contribute to each country's total emissions for the selected year: **${selectedYear}**.

```js
const topCountriesForBreakdown = yearData.slice(0, 12);

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
      range: ["#64748b", "#f59e0b", "#ef4444", "#8b5cf6", "#ec4899"]
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

---

## 🌐 Global Emissions Treemap

A treemap showing the relative size of emissions by country for year **${selectedYear}**. Larger blocks represent higher emissions. Hover to see detailed information.

```js
function emissionsTreemap({width} = {}) {
  const height = 650;

  // Prepare hierarchical data - show top 30 countries
  const topCountriesTreemap = yearData.slice(0, 30);
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
      range: ["#ef4444", "#f59e0b", "#22c55e", "#3b82f6", "#8b5cf6"]
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
