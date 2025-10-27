---
title: The Golden Century - Historical Gold Prices
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

.card {
  min-height: 750px;
}
</style>

<div class="hero">
  <h1>💰 The Golden Century</h1>
  <p>191 Years of Gold Price History (1833-2024)</p>
</div>

```js
// Load gold price data
const goldRaw = FileAttachment("data/annual-gold-prices.csv").csv({typed: true});
const goldData = (await goldRaw).map(d => ({
  year: +d.Date,
  price: +d.Price
})).filter(d => d.price > 0);

// Calculate key metrics
const latestPrice = goldData[goldData.length - 1].price;
const firstPrice = goldData[0].price;
const allTimeHigh = d3.max(goldData, d => d.price);
const allTimeHighYear = goldData.find(d => d.price === allTimeHigh).year;
const totalIncrease = ((latestPrice - firstPrice) / firstPrice * 100).toFixed(0);

// Find the gold standard era (fixed price period)
const goldStandardEnd = 1971; // Nixon ended gold standard
const priceIn1970 = goldData.find(d => d.year === 1970)?.price || 35;

// Categorize eras
const goldStandardData = goldData.filter(d => d.year <= 1970);
const modernEra = goldData.filter(d => d.year > 1970);
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
const selectedData = goldData.filter(d => d.year <= selectedYear);
const currentPrice = selectedData[selectedData.length - 1];
```

---

## 📈 The Great Transformation: 191 Years of Gold

This visualization tells the incredible story of gold - from over a century of stability to explosive modern growth. Use the timeline controls below the graph to explore the price at any point in history!

```js
function goldTimeline({width} = {}) {
  const isMobile = width < 640;
  return Plot.plot({
    width,
    height: isMobile ? 500 : 700,
    marginLeft: isMobile ? 60 : 90,
    marginRight: isMobile ? 20 : 120,
    marginBottom: isMobile ? 60 : 80,
    marginTop: 30,
    x: {
      label: "Year",
      grid: true,
      ticks: isMobile ? 5 : 15,
      tickFormat: "d"
    },
    y: {
      label: isMobile ? "Price (USD, log)" : "Price per Troy Ounce (USD, log scale)",
      grid: true,
      type: "log",
      ticks: [10, 20, 50, 100, 200, 500, 1000, 2000, 3000],
      labelAnchor: "center",
      labelArrow: "none"
    },
    marks: [
      // Background area fill with gradient
      Plot.areaY(selectedData, {
        x: "year",
        y: "price",
        fill: "url(#goldGradient)",
        fillOpacity: 0.5,
        curve: "catmull-rom"
      }),
      // Main gold line
      Plot.line(selectedData, {
        x: "year",
        y: "price",
        stroke: "#fbbf24",
        strokeWidth: 4,
        curve: "catmull-rom",
        tip: true,
        title: d => `${d.year}: $${d.price.toFixed(2)}`
      }),
      // Current year highlight
      Plot.dot([currentPrice], {
        x: "year",
        y: "price",
        r: 12,
        fill: "#fcd34d",
        stroke: "#ffffff",
        strokeWidth: 4
      }),
      Plot.text([currentPrice], {
        x: "year",
        y: "price",
        text: d => isMobile ? `$${d.price.toFixed(0)}` : `${d.year}: $${d.price.toFixed(2)}`,
        dx: isMobile ? -30 : -60,
        dy: -10,
        fill: "#fcd34d",
        fontSize: isMobile ? 11 : 13,
        fontWeight: "bold",
        textAnchor: "end"
      }),
      // 1971 Nixon Shock marker
      Plot.ruleX([1971], {
        stroke: "#dc2626",
        strokeWidth: isMobile ? 2 : 3,
        strokeDasharray: "6,4",
        opacity: 0.9
      }),
      Plot.text([{year: 1971, price: 1800}], {
        x: "year",
        y: "price",
        text: isMobile ? ["1971: Nixon"] : ["1971: Nixon Ends Gold Standard"],
        fill: "#dc2626",
        fontSize: isMobile ? 10 : 13,
        fontWeight: "bold",
        dx: isMobile ? -5 : -10,
        textAnchor: "end"
      }),
      // 2008 Financial Crisis marker
      Plot.ruleX([2008], {
        stroke: "#f59e0b",
        strokeWidth: isMobile ? 2 : 3,
        strokeDasharray: "6,4",
        opacity: 0.8
      }),
      Plot.text([{year: 2008, price: 900}], {
        x: "year",
        y: "price",
        text: isMobile ? ["2008: Crisis"] : ["2008: Financial Crisis"],
        fill: "#f59e0b",
        fontSize: isMobile ? 10 : 13,
        fontWeight: "bold",
        dx: isMobile ? -5 : -10,
        textAnchor: "end"
      }),
      // Gold Standard baseline
      Plot.ruleY([35], {
        stroke: "#22c55e",
        strokeWidth: 2,
        strokeDasharray: "3,3",
        opacity: 0.6
      }),
      ...(isMobile ? [] : [Plot.text([{year: 1900, price: 35}], {
        x: "year",
        y: "price",
        text: ["Gold Standard Era ($35) →"],
        fill: "#22c55e",
        fontSize: 13,
        dy: -8,
        textAnchor: "middle"
      })])
    ]
  });
}
```

<div class="card">
  ${resize((width) => goldTimeline({width}))}
</div>

```js
const years = d3.range(1833, 2025);
const selectedYear = view(Scrubber(years, {
  delay: 150,
  loop: false,
  initial: years.length - 1,
  autoplay: false,
  format: d => d
}));
```

<svg width="0" height="0">
  <defs>
    <linearGradient id="goldGradient" x1="0%" y1="0%" x2="0%" y2="100%">
      <stop offset="0%" style="stop-color:#fbbf24;stop-opacity:0.9" />
      <stop offset="50%" style="stop-color:#f59e0b;stop-opacity:0.6" />
      <stop offset="100%" style="stop-color:#78350f;stop-opacity:0.2" />
    </linearGradient>
  </defs>
</svg>

---

**Data Source:** [World Gold Council via DataHub.io](https://datahub.io/core/gold-prices)

**Period:** 1833-2024 (191 years)
**Unit:** US Dollars per Troy Ounce
