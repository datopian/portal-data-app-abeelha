---
title: Global Warming Effects Dashboard
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
  overflow-anchor: none !important;
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

body {
  overflow-anchor: none !important;
}

.card, .card > *, .card svg, .card figure {
  overflow-anchor: none !important;
  contain: layout style paint;
}

.card, .grid {
  max-width: 100% !important;
}

.chart-container {
  overflow-anchor: none !important;
  contain: layout style paint;
  min-height: 450px;
}

.chart-grid {
  overflow-anchor: none !important;
}

svg, svg * {
  overflow-anchor: none !important;
}

.stat-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1rem;
  margin: 2rem 0;
  overflow-anchor: none !important;
}

.stat-card {
  background: var(--theme-background-alt);
  border: 2px solid var(--theme-foreground-faint);
  border-radius: 12px;
  padding: 1.5rem;
  text-align: center;
  overflow-anchor: none !important;
}

.stat-value {
  font-size: 2.5rem;
  font-weight: 900;
  color: var(--theme-foreground-alt);
  margin: 0.5rem 0;
}

.stat-label {
  font-size: 0.9rem;
  color: var(--theme-foreground-muted);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.stat-change {
  font-size: 1rem;
  font-weight: 600;
  margin-top: 0.5rem;
}

.positive { color: #dc2626; }
.negative { color: #059669; }

.chart-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 1.5rem;
  margin: 2rem 0;
  overflow-anchor: none !important;
  contain: layout;
}

@media (max-width: 768px) {
  .chart-grid {
    grid-template-columns: 1fr;
  }

  .chart-container {
    min-height: 400px;
  }
}

.chart-container {
  background: white;
  border: 1px solid var(--theme-foreground-faint);
  border-radius: 8px;
  padding: 1rem;
}

.chart-container h3 {
  margin: 0 0 0.5rem 0;
  font-size: 1rem;
  font-weight: 700;
  color: var(--theme-foreground);
  overflow-anchor: none !important;
}

.chart-container > * {
  overflow-anchor: none !important;
}

form, form * {
  overflow-anchor: none !important;
}
</style>

<div class="hero">
  <h1>🌍 Global Warming Effects</h1>
  <p>Interactive dashboard showing the interconnected impacts of climate change across multiple environmental indicators</p>
</div>

```js
const epaRaw = FileAttachment("data/epa-sea-level.csv").csv({typed: true});
const csiroRaw = FileAttachment("data/CSIRO_Recons_gmsl_yr_2019.csv").csv({typed: true});
const glaciersRaw = FileAttachment("data/glaciers.csv").csv({typed: true});
const globalTempRaw = FileAttachment("data/global-temp-annual.csv").csv({typed: true});
```

```js
const csiroData = (await csiroRaw).map(d => ({
  year: +d.Time,
  value: +d.GMSL
})).filter(d => d.year >= 1880 && !isNaN(d.value));

const glaciersData = (await glaciersRaw).map(d => ({
  year: +d.Year,
  value: +d["Mean cumulative mass balance"]
})).filter(d => !isNaN(d.value) && d.year >= 1956);

const globalTempData = (await globalTempRaw)
  .filter(d => d.Source === "GISTEMP")
  .map(d => ({
    year: +d.Year,
    value: +d.Mean
  }))
  .filter(d => !isNaN(d.value) && d.year >= 1880);

const latestYear = 2019;
const baselineYear = 1880;
```

```js
const latestTemp = globalTempData.find(d => d.year === latestYear).value;
const baselineTemp = globalTempData.find(d => d.year === baselineYear).value;
const tempChange = latestTemp - baselineTemp;

const latestSeaLevel = csiroData.find(d => d.year === latestYear).value;
const baselineSeaLevel = csiroData.find(d => d.year === baselineYear).value;
const seaLevelChange = latestSeaLevel - baselineSeaLevel;

const latestGlacier = glaciersData.find(d => d.year === latestYear).value;
const baselineGlacier = glaciersData.find(d => d.year === 1956).value;
const glacierChange = latestGlacier - baselineGlacier;
```

<div class="stat-grid">
  <div class="stat-card">
    <div class="stat-label">Temperature Rise</div>
    <div class="stat-value positive">+${tempChange.toFixed(2)}°C</div>
    <div class="stat-change">Since ${baselineYear}</div>
  </div>
  <div class="stat-card">
    <div class="stat-label">Sea Level Rise</div>
    <div class="stat-value positive">+${seaLevelChange.toFixed(0)}mm</div>
    <div class="stat-change">Since ${baselineYear}</div>
  </div>
  <div class="stat-card">
    <div class="stat-label">Glacier Mass Loss</div>
    <div class="stat-value negative">${glacierChange.toFixed(1)}m</div>
    <div class="stat-change">Since 1956</div>
  </div>
</div>

## Climate Indicators Dashboard

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
const dashboardYears = d3.range(1880, 2020);
const selectedDashboardYear = view(Scrubber(dashboardYears, {
  delay: 100,
  loop: false,
  initial: dashboardYears.length - 1,
  autoplay: false,
  format: d => d
}));
```

```js
const filteredTemp = globalTempData.filter(d => d.year <= selectedDashboardYear);
const filteredSeaLevel = csiroData.filter(d => d.year <= selectedDashboardYear);
```

```js
function tripleAxisChart({width} = {}) {
  const isMobile = width < 640;
  const height = isMobile ? 400 : 420;
  const marginTop = 30;
  const marginRight = isMobile ? 55 : 70;
  const marginBottom = isMobile ? 90 : 80;
  const marginLeft = isMobile ? 55 : 70;

  const xScale = d3.scaleLinear()
    .domain([1880, 2019])
    .range([marginLeft, width - marginRight]);

  const yTempScale = d3.scaleLinear()
    .domain([d3.min(globalTempData, d => d.value), d3.max(globalTempData, d => d.value)])
    .range([height - marginBottom, marginTop])
    .nice();

  const ySeaLevelScale = d3.scaleLinear()
    .domain([d3.min(csiroData, d => d.value), d3.max(csiroData, d => d.value)])
    .range([height - marginBottom, marginTop])
    .nice();

  const lineTemp = d3.line()
    .x(d => xScale(d.year))
    .y(d => yTempScale(d.value))
    .curve(d3.curveCatmullRom);

  const lineSeaLevel = d3.line()
    .x(d => xScale(d.year))
    .y(d => ySeaLevelScale(d.value))
    .curve(d3.curveCatmullRom);

  const container = d3.create("div")
    .style("position", "relative");

  const svg = container.append("svg")
    .attr("width", width)
    .attr("height", height)
    .attr("viewBox", [0, 0, width, height])
    .attr("style", "max-width: 100%; height: auto; background: white; display: block;");

  const tooltip = container.append("div")
    .style("position", "absolute")
    .style("visibility", "hidden")
    .style("background", "white")
    .style("border", "1px solid #ccc")
    .style("border-radius", "4px")
    .style("padding", "8px")
    .style("font-size", "12px")
    .style("font-family", "system-ui, sans-serif")
    .style("pointer-events", "none")
    .style("box-shadow", "0 2px 4px rgba(0,0,0,0.1)")
    .style("z-index", "1000")
    .style("line-height", "1.5");

  const hoverCircleTemp = svg.append("circle")
    .attr("r", 5)
    .attr("fill", "#dc2626")
    .attr("stroke", "white")
    .attr("stroke-width", 2)
    .style("visibility", "hidden")
    .style("pointer-events", "none");

  const hoverCircleSea = svg.append("circle")
    .attr("r", 5)
    .attr("fill", "#059669")
    .attr("stroke", "white")
    .attr("stroke-width", 2)
    .style("visibility", "hidden")
    .style("pointer-events", "none");

  svg.append("g")
    .attr("class", "grid")
    .attr("transform", `translate(0,${height - marginBottom})`)
    .call(d3.axisBottom(xScale)
      .tickFormat("")
      .tickSize(-(height - marginTop - marginBottom)))
    .call(g => g.select(".domain").remove())
    .call(g => g.selectAll(".tick line")
      .attr("stroke", "#e5e7eb")
      .attr("stroke-opacity", 0.7));

  svg.append("g")
    .attr("class", "grid")
    .attr("transform", `translate(${marginLeft},0)`)
    .call(d3.axisLeft(yTempScale)
      .tickFormat("")
      .tickSize(-(width - marginLeft - marginRight)))
    .call(g => g.select(".domain").remove())
    .call(g => g.selectAll(".tick line")
      .attr("stroke", "#e5e7eb")
      .attr("stroke-opacity", 0.7));

  svg.append("g")
    .attr("transform", `translate(0,${height - marginBottom})`)
    .call(d3.axisBottom(xScale).tickFormat(d3.format("d")).ticks(isMobile ? 5 : 10).tickSize(0).tickPadding(8))
    .call(g => g.select(".domain").attr("stroke", "#ccc"))
    .call(g => g.selectAll("text").attr("font-size", isMobile ? "10px" : "11px"));

  svg.append("g")
    .attr("transform", `translate(${marginLeft},0)`)
    .call(d3.axisLeft(yTempScale).ticks(isMobile ? 5 : 7).tickSize(0).tickPadding(6).tickFormat(d => `${d}°C`))
    .call(g => g.select(".domain").attr("stroke", "#ccc"))
    .call(g => g.selectAll("text").attr("font-size", isMobile ? "10px" : "11px"));

  svg.append("g")
    .attr("transform", `translate(${width - marginRight},0)`)
    .call(d3.axisRight(ySeaLevelScale).ticks(isMobile ? 5 : 7).tickSize(0).tickPadding(6).tickFormat(d => `${d}mm`))
    .call(g => g.select(".domain").attr("stroke", "#ccc"))
    .call(g => g.selectAll("text").attr("font-size", isMobile ? "10px" : "11px"));

  svg.append("path")
    .datum(filteredTemp)
    .attr("fill", "none")
    .attr("stroke", "#dc2626")
    .attr("stroke-width", 3)
    .attr("d", lineTemp);

  svg.append("path")
    .datum(filteredSeaLevel)
    .attr("fill", "none")
    .attr("stroke", "#059669")
    .attr("stroke-width", 3)
    .attr("d", lineSeaLevel);

  svg.append("rect")
    .attr("x", marginLeft)
    .attr("y", marginTop)
    .attr("width", width - marginLeft - marginRight)
    .attr("height", height - marginTop - marginBottom)
    .attr("fill", "none")
    .attr("pointer-events", "all")
    .on("mousemove", function(event) {
      const [mx] = d3.pointer(event, svg.node());
      const year = Math.round(xScale.invert(mx));

      const tempData = filteredTemp.find(d => d.year === year);
      const seaData = filteredSeaLevel.find(d => d.year === year);

      if (tempData && seaData) {
        const tempX = xScale(tempData.year);
        const tempY = yTempScale(tempData.value);
        const seaY = ySeaLevelScale(seaData.value);

        hoverCircleTemp
          .attr("cx", tempX)
          .attr("cy", tempY)
          .style("visibility", "visible");

        hoverCircleSea
          .attr("cx", tempX)
          .attr("cy", seaY)
          .style("visibility", "visible");

        tooltip
          .html(`<strong>${year}</strong><br/>Temperature: ${tempData.value.toFixed(2)}°C<br/>Sea Level: ${seaData.value.toFixed(0)}mm`)
          .style("visibility", "visible");

        const tooltipNode = tooltip.node();
        const tooltipWidth = tooltipNode.offsetWidth;
        const tooltipHeight = tooltipNode.offsetHeight;

        const midY = (tempY + seaY) / 2;
        let left = tempX + 15;
        let top = midY - tooltipHeight / 2;

        if (left + tooltipWidth > width - marginRight) {
          left = tempX - tooltipWidth - 15;
        }
        if (top < marginTop) top = marginTop;
        if (top + tooltipHeight > height - marginBottom) top = height - marginBottom - tooltipHeight;

        tooltip
          .style("left", `${left}px`)
          .style("top", `${top}px`);
      }
    })
    .on("mouseout", () => {
      tooltip.style("visibility", "hidden");
      hoverCircleTemp.style("visibility", "hidden");
      hoverCircleSea.style("visibility", "hidden");
    });

  if (selectedDashboardYear >= 1880) {
    svg.append("line")
      .attr("x1", xScale(selectedDashboardYear))
      .attr("x2", xScale(selectedDashboardYear))
      .attr("y1", marginTop)
      .attr("y2", height - marginBottom)
      .attr("stroke", "#f59e0b")
      .attr("stroke-width", 2)
      .attr("stroke-dasharray", "4,4");

    const tempPoint = filteredTemp.find(d => d.year === selectedDashboardYear);
    if (tempPoint) {
      svg.append("circle")
        .attr("cx", xScale(tempPoint.year))
        .attr("cy", yTempScale(tempPoint.value))
        .attr("r", 6)
        .attr("fill", "#dc2626")
        .attr("stroke", "white")
        .attr("stroke-width", 2);
    }

    const seaLevelPoint = filteredSeaLevel.find(d => d.year === selectedDashboardYear);
    if (seaLevelPoint) {
      svg.append("circle")
        .attr("cx", xScale(seaLevelPoint.year))
        .attr("cy", ySeaLevelScale(seaLevelPoint.value))
        .attr("r", 6)
        .attr("fill", "#059669")
        .attr("stroke", "white")
        .attr("stroke-width", 2);
    }
  }

  const legend = svg.append("g")
    .attr("transform", `translate(${width / 2 - (isMobile ? 95 : 160)}, ${height - (isMobile ? 15 : 12)})`);

  legend.append("line")
    .attr("x1", 0)
    .attr("x2", isMobile ? 18 : 25)
    .attr("y1", 0)
    .attr("y2", 0)
    .attr("stroke", "#dc2626")
    .attr("stroke-width", 2.5);

  legend.append("text")
    .attr("x", isMobile ? 22 : 30)
    .attr("y", 4)
    .attr("fill", "currentColor")
    .attr("font-size", isMobile ? "10px" : "11px")
    .text(isMobile ? "Temperature" : "Temperature (°C)");

  legend.append("line")
    .attr("x1", isMobile ? 95 : 150)
    .attr("x2", isMobile ? 113 : 175)
    .attr("y1", 0)
    .attr("y2", 0)
    .attr("stroke", "#059669")
    .attr("stroke-width", 2.5);

  legend.append("text")
    .attr("x", isMobile ? 117 : 180)
    .attr("y", 4)
    .attr("fill", "currentColor")
    .attr("font-size", isMobile ? "10px" : "11px")
    .text(isMobile ? "Sea Level" : "Sea Level (mm)");

  return container.node();
}
```

```js
const tempRateData = globalTempData.slice(1).map((d, i) => ({
  year: d.year,
  rate: (d.value - globalTempData[i].value) * 10
})).filter(d => d.year >= 1880 && d.year <= 2019);

const filteredTempRateData = tempRateData.filter(d => d.year <= selectedDashboardYear);
```

```js
function rateOfChangeChart({width} = {}) {
  const isMobile = width < 640;

  return Plot.plot({
    width,
    height: isMobile ? 350 : 380,
    marginLeft: isMobile ? 50 : 70,
    marginRight: isMobile ? 50 : 70,
    marginBottom: isMobile ? 50 : 40,
    style: {
      fontSize: isMobile ? "11px" : "13px"
    },
    x: {
      label: "Year",
      grid: true,
      ticks: isMobile ? 5 : 10
    },
    y: {
      label: "Temperature Change (°C/decade)",
      grid: true
    },
    marks: [
      Plot.ruleY([0], {stroke: "#94a3b8", strokeDasharray: "2,2"}),
      Plot.line(filteredTempRateData, {
        x: "year",
        y: "rate",
        stroke: "#dc2626",
        strokeWidth: 2,
        curve: "catmull-rom"
      }),
      Plot.dot(filteredTempRateData, {
        x: "year",
        y: "rate",
        r: 3,
        fill: "transparent",
        stroke: "transparent",
        tip: true,
        title: d => `Year: ${d.year}\nRate: ${d.rate.toFixed(3)}°C/decade`
      }),
      Plot.ruleX([selectedDashboardYear], {stroke: "#f59e0b", strokeWidth: 2, strokeDasharray: "4,4"}),
      Plot.dot(filteredTempRateData.filter(d => d.year === selectedDashboardYear), {
        x: "year",
        y: "rate",
        fill: "#dc2626",
        stroke: "white",
        strokeWidth: 2,
        r: 6
      })
    ]
  });
}
```

```js
const glacierTempData = glaciersData
  .map(d => ({
    year: d.year,
    glacier: d.value,
    temp: globalTempData.find(t => t.year === d.year)?.value
  }))
  .filter(d => d.temp !== undefined);

const filteredGlacierTempData = glacierTempData.filter(d => d.year <= selectedDashboardYear);
```

```js
function glacierTempScatter({width} = {}) {
  const isMobile = width < 640;

  return Plot.plot({
    width,
    height: isMobile ? 350 : 380,
    marginLeft: isMobile ? 50 : 70,
    marginRight: isMobile ? 20 : 40,
    marginBottom: isMobile ? 50 : 60,
    style: {
      fontSize: isMobile ? "11px" : "13px"
    },
    x: {
      label: "Global Temperature Anomaly (°C)",
      grid: true
    },
    y: {
      label: "Glacier Mass Balance (m)",
      grid: true
    },
    color: {
      type: "linear",
      scheme: "YlOrRd",
      legend: true,
      label: "Year"
    },
    marks: [
      Plot.dot(filteredGlacierTempData, {
        x: "temp",
        y: "glacier",
        fill: "year",
        r: 4,
        tip: true,
        title: d => `Year: ${d.year}\nTemperature: ${d.temp.toFixed(2)}°C\nGlacier Mass: ${d.glacier.toFixed(2)}m`
      }),
      Plot.linearRegressionY(filteredGlacierTempData, {
        x: "temp",
        y: "glacier",
        stroke: "#1f2937",
        strokeWidth: 2,
        strokeDasharray: "4,4"
      }),
      Plot.dot(filteredGlacierTempData.filter(d => d.year === selectedDashboardYear), {
        x: "temp",
        y: "glacier",
        fill: "#f59e0b",
        stroke: "white",
        strokeWidth: 2,
        r: 8
      })
    ]
  });
}
```

```js
const decadeComparison = d3.rollup(
  globalTempData.filter(d => d.year >= 1880 && d.year < 2020),
  v => ({
    avgTemp: d3.mean(v, d => d.value),
    avgSeaLevel: d3.mean(csiroData.filter(s => s.year >= v[0].year && s.year < v[0].year + 10), d => d.value)
  }),
  d => Math.floor(d.year / 10) * 10
);

const decadeData = Array.from(decadeComparison, ([decade, values]) => ({
  decade: `${decade}s`,
  decadeNum: decade,
  temp: values.avgTemp,
  seaLevel: values.avgSeaLevel
})).sort((a, b) => a.decadeNum - b.decadeNum);

const filteredDecadeData = decadeData.filter(d => d.decadeNum <= Math.floor(selectedDashboardYear / 10) * 10);
```

```js
function decadeComparisonChart({width} = {}) {
  const isMobile = width < 640;
  const currentDecade = Math.floor(selectedDashboardYear / 10) * 10;

  return Plot.plot({
    width,
    height: isMobile ? 350 : 380,
    marginLeft: isMobile ? 50 : 70,
    marginRight: isMobile ? 20 : 40,
    marginBottom: isMobile ? 80 : 70,
    style: {
      fontSize: isMobile ? "11px" : "13px"
    },
    x: {
      label: null,
      tickRotate: -45
    },
    y: {
      label: "Average Temperature Anomaly (°C)",
      grid: true
    },
    color: {
      legend: false
    },
    marks: [
      Plot.barY(filteredDecadeData, {
        x: "decade",
        y: "temp",
        fill: d => d.decadeNum === currentDecade ? "#f59e0b" : "temp",
        fillOpacity: d => d.decadeNum === currentDecade ? 1 : 0.8,
        stroke: d => d.decadeNum === currentDecade ? "white" : "none",
        strokeWidth: 2,
        tip: true,
        title: d => `${d.decade}\nAvg Temp: ${d.temp.toFixed(3)}°C\nAvg Sea Level: ${d.seaLevel.toFixed(1)}mm`
      }),
      Plot.ruleY([0])
    ]
  });
}
```

<div class="chart-grid">
  <div class="chart-container">
    <h3>Combined Climate Indicators (1880-2019)</h3>
    ${resize((width) => tripleAxisChart({width}))}
  </div>

  <div class="chart-container">
    <h3>Rate of Change Analysis</h3>
    ${resize((width) => rateOfChangeChart({width}))}
  </div>

  <div class="chart-container">
    <h3>Glacier Mass Balance vs Global Temperature</h3>
    ${resize((width) => glacierTempScatter({width}))}
  </div>

  <div class="chart-container">
    <h3>Decade-by-Decade Comparison</h3>
    ${resize((width) => decadeComparisonChart({width}))}
  </div>
</div>

## Key Insights

- **Temperature Rise**: Global temperatures have increased by **${tempChange.toFixed(2)}°C** since 1880
- **Sea Level Rise**: Oceans have risen by **${seaLevelChange.toFixed(0)}mm** over the same period
- **Glacier Loss**: Reference glaciers have lost **${Math.abs(glacierChange).toFixed(1)} meters** of water equivalent since 1956
- **Acceleration**: The rate of change is increasing across all indicators
- **Correlation**: Strong positive relationship between temperature rise and both sea level increase and glacier mass loss

**Data Sources**: NASA GISTEMP, CSIRO, WGMS, EPA
