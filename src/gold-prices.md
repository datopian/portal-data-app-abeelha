---
title: The Golden Century - Historical Gold Prices
toc: false
---

<style>
.hero {
  text-align: center;
  padding: 3rem 2rem;
  background: linear-gradient(135deg, #fbbf24 0%, #f59e0b 30%, #d97706 60%, #92400e 100%);
  color: white;
  border-radius: 16px;
  margin-bottom: 2rem;
  box-shadow: 0 10px 40px rgba(251, 191, 36, 0.5);
  position: relative;
  overflow: hidden;
}

.hero::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, rgba(255,255,255,0.1) 0%, transparent 70%);
  animation: shimmer 3s infinite;
}

@keyframes shimmer {
  0%, 100% { transform: translate(0, 0); }
  50% { transform: translate(10%, 10%); }
}

.hero h1 {
  margin: 0;
  font-size: 3.5rem;
  font-weight: 900;
  color: white;
  text-shadow: 3px 3px 10px rgba(0,0,0,0.5);
  position: relative;
  z-index: 1;
}

.hero p {
  margin: 1rem 0 0;
  font-size: 1.3rem;
  opacity: 0.95;
  position: relative;
  z-index: 1;
}

.era-card {
  background: linear-gradient(135deg, #1e293b 0%, #0f172a 100%);
  padding: 1.5rem;
  border-radius: 12px;
  color: white;
  border-left: 6px solid;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.era-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 30px rgba(251, 191, 36, 0.3);
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
  background: linear-gradient(135deg, #fbbf24 0%, #f59e0b 60%, #d97706 100%);
  padding: 1rem 1rem;
  border-radius: 12px;
  box-shadow: -4px 0 20px rgba(217, 119, 6, 0.5);
  z-index: 1000;
  width: 160px;
  backdrop-filter: blur(10px);
  border: 2px solid rgba(251, 191, 36, 0.3);
}

.year-control-overlay h3 {
  color: white;
  margin: 0 0 0.5rem 0;
  font-size: 0.9rem;
  text-align: center;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
}

.year-control-overlay .year-display-overlay {
  font-size: 2rem;
  font-weight: bold;
  color: white;
  text-align: center;
  margin: 0.75rem 0 0.25rem 0;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.4);
  letter-spacing: 1px;
}

.year-control-overlay input[type="range"] {
  width: 100%;
  margin: 0.5rem 0;
  cursor: pointer;
  accent-color: #fcd34d;
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

svg text {
  fill: black !important;
}

[id^="plot-tip-"],
[role="tooltip"],
[aria-live="polite"] {
  color: black !important;
}

[id^="plot-tip-"] *,
[role="tooltip"] *,
[aria-live="polite"] * {
  color: black !important;
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

<div class="year-control-overlay">
  <h3>📅 Year Control</h3>

```js
const selectedYear = view(Inputs.range([1833, 2024], {
  step: 1,
  value: 2024
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
const selectedData = goldData.filter(d => d.year <= selectedYear);
const currentPrice = selectedData[selectedData.length - 1];
```

---

## 📈 The Great Transformation: 191 Years of Gold

This visualization tells the incredible story of gold - from over a century of stability to explosive modern growth. Select a year above to explore the price at any point in history!

```js
function goldTimeline({width} = {}) {
  return Plot.plot({
    width,
    height: 700,
    marginLeft: 90,
    marginRight: 30,
    marginBottom: 80,
    marginTop: 30,
    x: {
      label: "Year",
      grid: true,
      ticks: 15,
      tickFormat: "d"
    },
    y: {
      label: "Price per Troy Ounce (USD, log scale)",
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
        text: d => `${d.year}: $${d.price.toFixed(2)}`,
        dx: -60,
        dy: -10,
        fill: "#fcd34d",
        fontSize: 13,
        fontWeight: "bold",
        textAnchor: "end"
      }),
      // 1971 Nixon Shock marker
      Plot.ruleX([1971], {
        stroke: "#dc2626",
        strokeWidth: 3,
        strokeDasharray: "6,4",
        opacity: 0.9
      }),
      Plot.text([{year: 1971, price: 1800}], {
        x: "year",
        y: "price",
        text: ["1971: Nixon Ends Gold Standard"],
        fill: "#dc2626",
        fontSize: 13,
        fontWeight: "bold",
        dx: 10,
        textAnchor: "start"
      }),
      // 2008 Financial Crisis marker
      Plot.ruleX([2008], {
        stroke: "#f59e0b",
        strokeWidth: 3,
        strokeDasharray: "6,4",
        opacity: 0.8
      }),
      Plot.text([{year: 2008, price: 900}], {
        x: "year",
        y: "price",
        text: ["2008: Financial Crisis"],
        fill: "#f59e0b",
        fontSize: 13,
        fontWeight: "bold",
        dx: 10,
        textAnchor: "start"
      }),
      // Gold Standard baseline
      Plot.ruleY([35], {
        stroke: "#22c55e",
        strokeWidth: 2,
        strokeDasharray: "3,3",
        opacity: 0.6
      }),
      Plot.text([{year: 1900, price: 35}], {
        x: "year",
        y: "price",
        text: ["Gold Standard Era ($35) →"],
        fill: "#22c55e",
        fontSize: 13,
        dy: -8,
        textAnchor: "middle"
      })
    ]
  });
}
```

<div class="card">
  ${resize((width) => goldTimeline({width}))}
</div>

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
