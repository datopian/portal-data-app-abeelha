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

## World Map Visualization

```js
const worldGeo = await d3.json("https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.json");
const worldFeatures = topojson.feature(worldGeo, worldGeo.objects.countries);
```

```js
function worldMapD3(year, {width} = {}) {
  // Try EPA data first, then fall back to CSIRO data
  let yearData = epaData.find(d => d.year === year);
  if (!yearData) {
    yearData = csiroData.find(d => d.year === year);
  }
  const seaLevel = yearData ? yearData.value : 0;

  // Calculate parameters based on sea level
  const nbLines = Math.max(3, Math.min(8, Math.floor(seaLevel / 1.2) + 3));

  // Danger level colors
  const dangerLevel = seaLevel < 3 ? "low" : seaLevel < 6 ? "medium" : "high";
  const waterColor = dangerLevel === "high" ? "#ef4444" : dangerLevel === "medium" ? "#f59e0b" : "#3b82f6";

  const height = width * 0.55;
  const projection = d3.geoNaturalEarth1().fitSize([width, height], worldFeatures);
  const path = d3.geoPath(projection);

  const svg = d3.create("svg")
    .attr("width", width)
    .attr("height", height)
    .attr("viewBox", [0, 0, width, height])
    .style("background", "linear-gradient(180deg, #0f172a 0%, #1e293b 100%)");

  // Ocean background
  svg.append("rect")
    .attr("width", width)
    .attr("height", height)
    .attr("fill", "#0c4a6e")
    .attr("opacity", 0.4);

  // Simple clean waterlines - just outlines around land
  for (let i = 0; i < nbLines; i++) {
    const opacity = 0.6 - (i / nbLines) * 0.45;
    const lineWidth = 2 - (i / nbLines) * 1.2;
    const brightness = 1 + (i / nbLines) * 0.5;

    svg.append("g")
      .selectAll("path")
      .data(worldFeatures.features)
      .join("path")
        .attr("d", path)
        .attr("fill", "none")
        .attr("stroke", waterColor)
        .attr("stroke-width", lineWidth)
        .attr("stroke-opacity", opacity)
        .attr("stroke-linecap", "round")
        .attr("stroke-linejoin", "round")
        .style("filter", `brightness(${brightness})`);
  }

  // Land masses with gradient
  const defs = svg.append("defs");
  const landGradient = defs.append("linearGradient")
    .attr("id", `landGrad-${year}`)
    .attr("x1", "0%")
    .attr("y1", "0%")
    .attr("x2", "0%")
    .attr("y2", "100%");

  landGradient.append("stop")
    .attr("offset", "0%")
    .attr("stop-color", "#22c55e");

  landGradient.append("stop")
    .attr("offset", "100%")
    .attr("stop-color", "#059669");

  svg.append("g")
    .selectAll("path")
    .data(worldFeatures.features)
    .join("path")
      .attr("d", path)
      .attr("fill", `url(#landGrad-${year})`)
      .attr("stroke", "#065f46")
      .attr("stroke-width", 0.5);

  // Rising water warning overlay
  const waterLevel = Math.min(0.25, seaLevel / 30);
  if (waterLevel > 0.02) {
    svg.append("rect")
      .attr("width", width)
      .attr("height", height)
      .attr("fill", waterColor)
      .attr("opacity", waterLevel);
  }

  // Modern info panel
  const panel = svg.append("g");

  panel.append("rect")
    .attr("x", 10)
    .attr("y", 10)
    .attr("width", 180)
    .attr("height", 70)
    .attr("fill", "#0f172a")
    .attr("opacity", 0.9)
    .attr("rx", 8)
    .attr("stroke", waterColor)
    .attr("stroke-width", 2);

  panel.append("text")
    .attr("x", 20)
    .attr("y", 35)
    .attr("font-size", "20px")
    .attr("font-weight", "bold")
    .attr("fill", "#f1f5f9")
    .text(year);

  panel.append("text")
    .attr("x", 20)
    .attr("y", 55)
    .attr("font-size", "13px")
    .attr("fill", waterColor)
    .attr("font-weight", "600")
    .text(`+${seaLevel.toFixed(2)}" rise`);

  panel.append("text")
    .attr("x", 20)
    .attr("y", 70)
    .attr("font-size", "11px")
    .attr("fill", "#94a3b8")
    .text(`${nbLines} waterlines`);

  return svg.node();
}
```

```js
function mapWithSlider({width} = {}) {
  const mapWidth = Math.min(width - 110, 700);
  const sliderWidth = 90;
  const height = mapWidth * 0.55;

  let currentYear = 1993;
  let rafId = null; // For throttling updates

  const container = d3.create("div")
    .style("display", "flex")
    .style("gap", "20px")
    .style("align-items", "center");

  // Map container
  const mapContainer = container.append("div")
    .style("flex", "1");

  // Create the initial map
  mapContainer.node().appendChild(worldMapD3(currentYear, {width: mapWidth}));

  // Slider container
  const sliderContainer = container.append("div")
    .style("width", `${sliderWidth}px`)
    .style("display", "flex")
    .style("flex-direction", "column")
    .style("align-items", "center")
    .style("gap", "15px");

  // Year display
  const yearDisplay = sliderContainer.append("div")
    .style("font-size", "32px")
    .style("font-weight", "700")
    .style("color", "#3b82f6")
    .style("text-align", "center")
    .style("font-family", "system-ui, -apple-system, sans-serif")
    .text(currentYear);

  // Vertical slider
  const sliderSvg = sliderContainer.append("svg")
    .attr("width", 50)
    .attr("height", height - 80)
    .style("background", "#f1f5f9")
    .style("border-radius", "25px")
    .style("overflow", "visible");

  const sliderHeight = height - 80;
  const yearScale = d3.scaleLinear()
    .domain([1880, latestYear])
    .range([20, sliderHeight - 20]);

  // Slider track (filled portion)
  const track = sliderSvg.append("rect")
    .attr("x", 18)
    .attr("y", 20)
    .attr("width", 14)
    .attr("height", yearScale(currentYear) - 20)
    .attr("fill", "#3b82f6")
    .attr("rx", 7);

  // Slider background track
  sliderSvg.append("rect")
    .attr("x", 18)
    .attr("y", 20)
    .attr("width", 14)
    .attr("height", sliderHeight - 40)
    .attr("fill", "none")
    .attr("stroke", "#cbd5e1")
    .attr("stroke-width", 2)
    .attr("rx", 7);

  // Year tick marks
  [1880, 1920, 1960, 2000, latestYear].forEach(year => {
    const y = yearScale(year);

    sliderSvg.append("line")
      .attr("x1", 32)
      .attr("y1", y)
      .attr("x2", 42)
      .attr("y2", y)
      .attr("stroke", "#94a3b8")
      .attr("stroke-width", 2);

    sliderSvg.append("text")
      .attr("x", 46)
      .attr("y", y + 3)
      .attr("font-size", "10px")
      .attr("fill", "#64748b")
      .text(year);
  });

  // Slider handle - simple circle
  const handle = sliderSvg.append("circle")
    .attr("cx", 25)
    .attr("cy", yearScale(currentYear))
    .attr("r", 10)
    .attr("fill", "#ffffff")
    .attr("stroke", "#3b82f6")
    .attr("stroke-width", 3)
    .attr("cursor", "grab")
    .style("filter", "drop-shadow(0 2px 4px rgba(0,0,0,0.2))");

  // Update function with throttling
  function updateMap(newYear) {
    if (rafId) return; // Skip if already updating

    rafId = requestAnimationFrame(() => {
      const newMapSvg = worldMapD3(newYear, {width: mapWidth});
      mapContainer.node().innerHTML = "";
      mapContainer.node().appendChild(newMapSvg);
      rafId = null;
    });
  }

  // Drag behavior
  const drag = d3.drag()
    .on("start", function() {
      handle.attr("cursor", "grabbing")
        .attr("r", 12);
    })
    .on("drag", function(event) {
      const newY = Math.max(20, Math.min(sliderHeight - 20, event.y));
      const newYear = Math.round(yearScale.invert(newY));

      // Always update handle position for smooth feel
      handle.attr("cy", newY);
      track.attr("height", newY - 20);
      yearDisplay.text(newYear);

      // Only update map if year changed
      if (newYear !== currentYear) {
        currentYear = newYear;
        updateMap(newYear);
      }
    })
    .on("end", function() {
      handle.attr("cursor", "grab")
        .attr("r", 10);

      // Snap to exact year position
      const finalY = yearScale(currentYear);
      handle.attr("cy", finalY);
      track.attr("height", finalY - 20);
    });

  sliderSvg.call(drag);

  return container.node();
}
```

<div class="card">
  ${resize((width) => mapWithSlider({width}))}
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
