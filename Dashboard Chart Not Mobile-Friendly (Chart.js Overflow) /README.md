# Dashboard Chart Not Mobile-Friendly (Chart.js Overflow) 

## Description

When viewing the dashboard on a mobile device or in a small browser window, the Latest Presences chart (Chart.js) in the admin dashboard overflows its card container.  
This causes part of the chart and its legend to be hidden or clipped, impacting usability and readability on mobile screens.

## Screenshot

![Dashboard Chart Overflow](./path-to-your-screenshot.png)

## Root Cause

- The Chart.js canvas has a fixed width and/or height and does not scale responsively within its parent container.
- The parent card/container and legend do not have appropriate responsive styles for mobile breakpoints.
- Chart.js options may not have `responsive: true` and `maintainAspectRatio: false`, further limiting flexibility.

## Solution

### 1. **Add Responsive CSS**

Add this to your main CSS (e.g., `public/assets/css/app.css` or your main style file):

```css
@media (max-width: 576px) {
    .card .card-body canvas {
        width: 100% !important;
        height: 160px !important;
        max-width: 100vw;
        display: block;
    }
    .card .card-body .chartjs-legend {
        font-size: 12px !important;
        text-align: left;
    }
}
````

Optionally, set the canvas directly:

```css
#presence {
    width: 100% !important;
    height: 200px !important;
    max-width: 100vw;
}
```

### 2. **Update Chart.js Initialization**

Ensure your Chart.js initialization includes:

```js
const presenceChart = new Chart(ctxLine, {
    type: 'line',
    data: { /* ... */ },
    options: {
        responsive: true,
        maintainAspectRatio: false,
        // ...
    }
});
```

### 3. **Test Responsiveness**

* Use mobile devices or browser DevTools to verify that the chart and legend are visible and adapt properly to smaller screens.

## Result

With these changes, the dashboard chart will become fully mobile-friendly, ensuring data remains accessible and the interface remains clean across devices—without modifying your Blade template.

---

## References

* [Chart.js Responsive Docs](https://www.chartjs.org/docs/latest/configuration/responsive.html)
* [CSS Media Queries MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries/Using_media_queries)

---

**If you encounter a chart or UI element that overflows its card/container on mobile, always check CSS breakpoints and Chart.js responsive settings!**
