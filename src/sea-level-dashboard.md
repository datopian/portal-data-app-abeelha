---
title: Global Sea Level Rise Dashboard
toc: false
---

<style>
.hero {
  text-align: center;
  padding: 2rem;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border-radius: 12px;
  margin-bottom: 2rem;
}

.hero h1 {
  margin: 0;
  color: white;
}

.year-display {
  font-size: 3rem;
  font-weight: bold;
  color: #4f46e5;
  text-align: center;
  margin: 1rem 0;
}

/* Smooth transitions for map elements */
svg rect, svg path, svg circle, svg text {
  transition: opacity 0.5s ease-in-out, fill 0.5s ease-in-out;
}
</style>

<div class="hero">
  <h1>🌊 Global Sea Level Rise Dashboard</h1>
  <p>Tracking 140+ years of sea level changes</p>
</div>

```js
// Load data
const epaRaw = FileAttachment("data/epa-sea-level.csv").csv({typed: true});
const csiroRaw = FileAttachment("data/CSIRO_Recons_gmsl_yr_2019.csv").csv({typed: true});
const satRaw = FileAttachment("data/CSIRO_Alt_yearly.csv").csv({typed: true});
```

```js
// Normalize data
const epaData = (await epaRaw).map(d => ({
  year: +d.Year,
  value: +d["CSIRO Adjusted Sea Level"] || 0,
  lower: +d["Lower Error Bound"] || 0,
  upper: +d["Upper Error Bound"] || 0,
  source: "EPA/CSIRO"
})).filter(d => d.year >= 1880 && d.year <= 2023);

const csiroData = (await csiroRaw).map(d => ({
  year: +d.Time,
  value: +d.GMSL / 25.4,
  source: "CSIRO"
})).filter(d => d.year >= 1880);

const satData = (await satRaw).map(d => ({
  year: +d.Time,
  value: +d["GMSL (yearly)"] / 25.4,
  source: "Satellite"
})).filter(d => d.year >= 1993);

const allData = [...epaData, ...csiroData, ...satData];
```

```js
// Get the latest year with actual data
const latestYear = Math.max(...csiroData.map(d => d.year));

// Controls
const selectedYear = view(Inputs.range([1880, latestYear], {
  label: "Year",
  step: 1,
  value: 1993
}));
```

```js
const selectedDataset = view(Inputs.select(
  ["All", "EPA/CSIRO", "CSIRO", "Satellite"],
  {label: "Dataset", value: "All"}
));
```

<div class="year-display">${selectedYear}</div>

```js
// Filter data
const filteredData = allData.filter(d => {
  if (selectedDataset === "All") return d.year <= selectedYear;
  if (selectedDataset === "EPA/CSIRO") return d.source === "EPA/CSIRO" && d.year <= selectedYear;
  if (selectedDataset === "CSIRO") return d.source === "CSIRO" && d.year <= selectedYear;
  if (selectedDataset === "Satellite") return d.source === "Satellite" && d.year <= selectedYear;
  return false;
});
```

```js
// Current stats
const currentEPA = epaData.find(d => d.year === selectedYear);
const currentCSIRO = csiroData.find(d => d.year === selectedYear);
const currentSat = satData.find(d => d.year === selectedYear);
```

<div class="grid grid-cols-3">
  <div class="card">
    <h3>EPA/CSIRO</h3>
    <span class="big">${currentEPA ? currentEPA.value.toFixed(2) : "N/A"}</span>
    <p>inches</p>
  </div>
  <div class="card">
    <h3>CSIRO</h3>
    <span class="big">${currentCSIRO ? currentCSIRO.value.toFixed(2) : "N/A"}</span>
    <p>inches</p>
  </div>
  <div class="card">
    <h3>Satellite</h3>
    <span class="big">${currentSat ? currentSat.value.toFixed(2) : "N/A"}</span>
    <p>inches</p>
  </div>
</div>

## 📈 Statistical Insights

```js
// Calculate comprehensive statistics
const historicalAvg = d3.mean(csiroData.filter(d => d.year >= 1900 && d.year <= 1990), d => d.value);
const recentAvg = d3.mean(csiroData.filter(d => d.year >= 1993 && d.year <= latestYear), d => d.value);
const currentValue = currentCSIRO ? currentCSIRO.value : 0;

// Rate calculations
const hist1900Data = csiroData.filter(d => d.year >= 1900 && d.year <= 1990);
const hist1900Rate = hist1900Data.length > 1
  ? ((hist1900Data[hist1900Data.length - 1].value - hist1900Data[0].value) / (hist1900Data.length - 1)) * 10
  : 0;

const recent1993Data = csiroData.filter(d => d.year >= 1993 && d.year <= latestYear);
const recent1993Rate = recent1993Data.length > 1
  ? ((recent1993Data[recent1993Data.length - 1].value - recent1993Data[0].value) / (recent1993Data.length - 1)) * 10
  : 0;

// Acceleration
const acceleration = hist1900Rate > 0 ? ((recent1993Rate - hist1900Rate) / hist1900Rate * 100) : 0;

// Percentile of current value
const sortedValues = csiroData.map(d => d.value).sort((a, b) => a - b);
const percentile = (sortedValues.filter(v => v <= currentValue).length / sortedValues.length * 100);

// Simple linear extrapolation to 2050 (from latest year)
const latestValue = csiroData[csiroData.length - 1].value;
const yearsToProject = 2050 - latestYear;
const projected2050 = latestValue + (recent1993Rate / 10 * yearsToProject);

// Worst case (accelerated) projection
const projected2050Accelerated = latestValue + (recent1993Rate * 1.5 / 10 * yearsToProject);

// Calculate absolute difference vs historical avg
const vsHistorical = currentValue - historicalAvg;
```

<div class="grid grid-cols-4" style="margin-bottom: 2rem;">
  <div class="card" style="background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%); color: white;">
    <h3 style="color: white; font-size: 0.9rem;">Current vs Historical</h3>
    <span class="big" style="color: white;">${vsHistorical >= 0 ? '+' : ''}${vsHistorical.toFixed(1)}"</span>
    <p style="color: rgba(255,255,255,0.8); font-size: 0.85rem;">vs 1900-1990 avg (${historicalAvg.toFixed(1)}")</p>
  </div>
  <div class="card" style="background: linear-gradient(135deg, #f59e0b 0%, #d97706 100%); color: white;">
    <h3 style="color: white; font-size: 0.9rem;">Rate Acceleration</h3>
    <span class="big" style="color: white;">${acceleration.toFixed(0)}%</span>
    <p style="color: rgba(255,255,255,0.8); font-size: 0.85rem;">faster than 20th century</p>
  </div>
  <div class="card" style="background: linear-gradient(135deg, #8b5cf6 0%, #7c3aed 100%); color: white;">
    <h3 style="color: white; font-size: 0.9rem;">Current Percentile</h3>
    <span class="big" style="color: white;">${percentile.toFixed(0)}th</span>
    <p style="color: rgba(255,255,255,0.8); font-size: 0.85rem;">of all recorded values</p>
  </div>
  <div class="card" style="background: linear-gradient(135deg, #dc2626 0%, #b91c1c 100%); color: white;">
    <h3 style="color: white; font-size: 0.9rem;">2050 Projection</h3>
    <span class="big" style="color: white;">${projected2050.toFixed(1)}"</span>
    <p style="color: rgba(255,255,255,0.8); font-size: 0.85rem;">current trend (${projected2050Accelerated.toFixed(1)}" if accelerates)</p>
  </div>
</div>

## Sea Level Change Over Time

```js
function seaLevelChart(data, {width} = {}) {
  return Plot.plot({
    title: "Global Mean Sea Level Rise",
    width,
    height: 400,
    marginLeft: 60,
    x: {label: "Year", grid: true, domain: [1880, latestYear]},
    y: {label: "Sea Level Change (inches)", grid: true},
    color: {
      legend: true,
      domain: ["EPA/CSIRO", "CSIRO", "Satellite"],
      range: ["#4f46e5", "#059669", "#dc2626"]
    },
    marks: [
      // Uncertainty band for EPA
      Plot.areaY(
        data.filter(d => d.source === "EPA/CSIRO" && d.lower !== undefined),
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
        title: d => `${d.source} (${d.year}): ${d.value.toFixed(2)} inches`
      })),
      Plot.ruleY([0])
    ]
  });
}
```

<div class="card">
  ${resize((width) => seaLevelChart(filteredData, {width}))}
</div>

---

## 🌊 Horizon View - Compact Multi-Dataset Comparison

```js
function horizonChart({width} = {}) {
  const bands = 4;
  const bandHeight = 30;
  const height = bandHeight * bands;
  const datasets = ["EPA/CSIRO", "CSIRO", "Satellite"];

  // Group data by source
  const epaOnly = epaData.filter(d => d.value > 0);
  const csiroOnly = csiroData.filter(d => d.value > 0);
  const satOnly = satData.filter(d => d.value > 0);

  const dataBySource = {
    "EPA/CSIRO": epaOnly,
    "CSIRO": csiroOnly,
    "Satellite": satOnly
  };

  // Calculate max value for scaling
  const maxValue = d3.max([...epaOnly, ...csiroOnly, ...satOnly], d => d.value);

  return Plot.plot({
    width,
    height: height * 3 + 80,
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
      // Current year marker
      Plot.ruleX([selectedYear], {
        stroke: "#f59e0b",
        strokeWidth: 2,
        strokeDasharray: "4,4"
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
        fill: d => d === "EPA/CSIRO" ? "#4f46e5" : d === "CSIRO" ? "#059669" : "#dc2626",
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
          title: d => `${d.source}\n${d.year}: ${d.value.toFixed(2)} inches`,
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

## 📅 Calendar Heatmap - Annual Rate of Change Patterns

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
      label: "Annual Change (inches)"
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
        title: d => `${d.year}\nRate: ${d.rate >= 0 ? '+' : ''}${d.rate.toFixed(3)} in/year\nTotal: ${d.absValue.toFixed(2)} inches`
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
      }),
      // Current year highlight
      Plot.frame({
        fy: calendarData.filter(d => d.year === selectedYear).map(d => d.decadeLabel),
        stroke: "#f59e0b",
        strokeWidth: 3,
        inset: -2
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
    y: {label: "Average Sea Level (inches)", grid: true},
    marks: [
      Plot.barY(data, {
        x: "decade",
        y: "value",
        fill: d => d.value < 1 ? "#059669" : d.value < 3 ? "#f59e0b" : "#dc2626",
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

## 🔄 Connected Scatterplot - Sea Level vs Rate of Change

```js
// Prepare scatterplot data with rate of change
const scatterData = csiroData.slice(1).map((d, i) => {
  const prevValue = csiroData[i].value;
  const rate = (d.value - prevValue) * 10; // Rate per decade
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
      label: "Sea Level Change (inches)",
      nice: true
    },
    y: {
      label: "Rate of Change (inches/decade)",
      nice: true
    },
    color: {
      type: "linear",
      domain: [1880, latestYear],
      scheme: "Spectral",
      reverse: false,
      legend: true,
      label: "Year"
    },
    marks: [
      // Connected path showing temporal progression
      Plot.line(scatterData, {
        x: "value",
        y: "rate",
        stroke: "year",
        strokeWidth: 2,
        curve: "catmull-rom"
      }),
      // Dots for each year
      Plot.dot(scatterData, {
        x: "value",
        y: "rate",
        fill: "year",
        r: 3,
        stroke: "white",
        strokeWidth: 1,
        tip: true,
        title: d => `${d.year}\nSea Level: ${d.value.toFixed(2)} in\nRate: ${d.rate >= 0 ? '+' : ''}${d.rate.toFixed(2)} in/decade`
      }),
      // Arrows showing direction (every 10 years)
      Plot.arrow(
        scatterData.filter((d, i) => i % 10 === 0 && i < scatterData.length - 1),
        {
          x1: "value",
          y1: "rate",
          x2: (d, i, data) => {
            const nextIndex = scatterData.findIndex(item => item.year === d.year + 10);
            return nextIndex >= 0 ? scatterData[nextIndex].value : d.value;
          },
          y2: (d, i, data) => {
            const nextIndex = scatterData.findIndex(item => item.year === d.year + 10);
            return nextIndex >= 0 ? scatterData[nextIndex].rate : d.rate;
          },
          stroke: "year",
          strokeWidth: 1.5,
          headLength: 8,
          headAngle: 45
        }
      ),
      // Highlight selected year
      Plot.dot(
        scatterData.filter(d => d.year === selectedYear),
        {
          x: "value",
          y: "rate",
          r: 8,
          fill: "#f59e0b",
          stroke: "#ffffff",
          strokeWidth: 3
        }
      ),
      // Reference lines
      Plot.ruleY([0], {stroke: "#94a3b8", strokeDasharray: "4,4"}),
      Plot.ruleX([0], {stroke: "#94a3b8", strokeDasharray: "4,4"})
    ]
  });
}
```

<div class="card">
  ${resize((width) => connectedScatterplot({width}))}
</div>

---

## 🌀 Spiral Timeline - 140 Years in Circular View

```js
function spiralTimeline({width} = {}) {
  const size = Math.min(width, 700);
  const centerX = size / 2;
  const centerY = size / 2;
  const maxRadius = size / 2 - 60;

  // Prepare spiral data
  const spiralData = csiroData.filter(d => d.year >= 1880 && d.year <= latestYear);
  const totalYears = spiralData.length;

  const svg = d3.create("svg")
    .attr("width", size)
    .attr("height", size)
    .attr("viewBox", [0, 0, size, size])
    .style("background", "#0f172a");

  // Create Archimedean spiral
  const spiralPath = d3.path();
  spiralData.forEach((d, i) => {
    const t = i / totalYears;
    const angle = t * Math.PI * 8; // 4 full rotations
    const radius = t * maxRadius;
    const x = centerX + radius * Math.cos(angle);
    const y = centerY + radius * Math.sin(angle);

    if (i === 0) {
      spiralPath.moveTo(x, y);
    } else {
      spiralPath.lineTo(x, y);
    }
  });

  // Draw spiral guide
  svg.append("path")
    .attr("d", spiralPath.toString())
    .attr("fill", "none")
    .attr("stroke", "#1e293b")
    .attr("stroke-width", 20)
    .attr("stroke-linecap", "round");

  // Color scale for values
  const colorScale = d3.scaleSequential(d3.interpolateRdYlBu)
    .domain([d3.max(spiralData, d => d.value), d3.min(spiralData, d => d.value)]);

  // Draw data points on spiral
  const segments = svg.append("g")
    .selectAll("circle")
    .data(spiralData)
    .join("circle")
      .attr("cx", (d, i) => {
        const t = i / totalYears;
        const angle = t * Math.PI * 8;
        const radius = t * maxRadius;
        return centerX + radius * Math.cos(angle);
      })
      .attr("cy", (d, i) => {
        const t = i / totalYears;
        const angle = t * Math.PI * 8;
        const radius = t * maxRadius;
        return centerY + radius * Math.sin(angle);
      })
      .attr("r", 3)
      .attr("fill", d => colorScale(d.value))
      .attr("stroke", "white")
      .attr("stroke-width", 0.5)
      .style("cursor", "pointer");

  // Add decade markers
  const decades = spiralData.filter(d => d.year % 10 === 0);
  svg.append("g")
    .selectAll("text")
    .data(decades)
    .join("text")
      .attr("x", (d, i) => {
        const index = spiralData.findIndex(item => item.year === d.year);
        const t = index / totalYears;
        const angle = t * Math.PI * 8;
        const radius = t * maxRadius + 20;
        return centerX + radius * Math.cos(angle);
      })
      .attr("y", (d, i) => {
        const index = spiralData.findIndex(item => item.year === d.year);
        const t = index / totalYears;
        const angle = t * Math.PI * 8;
        const radius = t * maxRadius + 20;
        return centerY + radius * Math.sin(angle);
      })
      .attr("text-anchor", "middle")
      .attr("font-size", "11px")
      .attr("font-weight", "bold")
      .attr("fill", "#f1f5f9")
      .text(d => d.year);

  // Add title
  svg.append("text")
    .attr("x", centerX)
    .attr("y", 30)
    .attr("text-anchor", "middle")
    .attr("font-size", "18px")
    .attr("font-weight", "bold")
    .attr("fill", "#f1f5f9")
    .text("140-Year Spiral");

  // Add legend
  const legendWidth = 200;
  const legendHeight = 10;
  const legendX = (size - legendWidth) / 2;
  const legendY = size - 30;

  const legendScale = d3.scaleLinear()
    .domain([0, legendWidth])
    .range([d3.max(spiralData, d => d.value), d3.min(spiralData, d => d.value)]);

  svg.append("g")
    .selectAll("rect")
    .data(d3.range(legendWidth))
    .join("rect")
      .attr("x", d => legendX + d)
      .attr("y", legendY)
      .attr("width", 1)
      .attr("height", legendHeight)
      .attr("fill", d => colorScale(legendScale(d)));

  svg.append("text")
    .attr("x", legendX)
    .attr("y", legendY - 5)
    .attr("font-size", "10px")
    .attr("fill", "#94a3b8")
    .text(`${d3.max(spiralData, d => d.value).toFixed(1)}"`);

  svg.append("text")
    .attr("x", legendX + legendWidth)
    .attr("y", legendY - 5)
    .attr("text-anchor", "end")
    .attr("font-size", "10px")
    .attr("fill", "#94a3b8")
    .text(`${d3.min(spiralData, d => d.value).toFixed(1)}"`);

  // Add interactivity
  segments.on("mouseenter", function(event, d) {
    d3.select(this)
      .transition()
      .duration(150)
      .attr("r", 6)
      .attr("stroke-width", 2);

    // Show tooltip
    const tooltip = svg.append("g").attr("class", "tooltip");

    tooltip.append("rect")
      .attr("x", centerX - 60)
      .attr("y", centerY - 40)
      .attr("width", 120)
      .attr("height", 50)
      .attr("fill", "#1e293b")
      .attr("stroke", "#3b82f6")
      .attr("stroke-width", 2)
      .attr("rx", 6);

    tooltip.append("text")
      .attr("x", centerX)
      .attr("y", centerY - 20)
      .attr("text-anchor", "middle")
      .attr("font-size", "16px")
      .attr("font-weight", "bold")
      .attr("fill", "#f1f5f9")
      .text(d.year);

    tooltip.append("text")
      .attr("x", centerX)
      .attr("y", centerY - 2)
      .attr("text-anchor", "middle")
      .attr("font-size", "14px")
      .attr("fill", "#3b82f6")
      .text(`+${d.value.toFixed(2)} inches`);
  })
  .on("mouseleave", function() {
    d3.select(this)
      .transition()
      .duration(150)
      .attr("r", 3)
      .attr("stroke-width", 0.5);

    svg.select(".tooltip").remove();
  });

  return svg.node();
}
```

<div class="card">
  ${resize((width) => spiralTimeline({width}))}
</div>

---

## About This Data

```js
// Calculate total rise correctly
const latestCSIRO = csiroData[csiroData.length - 1];
const totalRise = latestCSIRO ? latestCSIRO.value.toFixed(1) : "N/A";
```

This dashboard visualizes global mean sea level rise from 1880 to ${latestYear} using three authoritative datasets:

- **EPA/CSIRO** (1880-2013): Combined Environmental Protection Agency and CSIRO data
- **CSIRO Reconstructed** (1880-2019): Historical tide gauge records
- **Satellite Altimetry** (1993-2020): High-precision satellite measurements

All measurements show change relative to the 1900 baseline.

## Key Findings

- Sea levels have risen approximately **${totalRise} inches** since 1880
- The rate of rise has **accelerated** from ~0.6 in/decade (1900-1990) to ~1.2 in/decade (1993-${latestYear})
- Multiple independent datasets show **consistent trends**
- Satellite measurements (1993+) show the highest precision and confirm acceleration

---

**Note:** This shows global mean sea level. Local changes vary due to land subsidence, ocean currents, and regional climate patterns.
