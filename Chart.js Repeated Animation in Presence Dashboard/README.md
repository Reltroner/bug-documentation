# 🛠 Fix: Chart.js Repeated Animation in Presence Dashboard

This fix addresses the issue where the "Latest Presences" bar chart on the dashboard continuously animates every few seconds, even when the data hasn't changed.

---

## 🐞 Problem

The following script refreshes the chart every 5 seconds using `setInterval`:

```javascript
updateData();
setInterval(updateData, 5000);
````

Each call to `updateData()` triggers a `myBar.update()`, causing Chart.js to re-render the chart and re-animate all bars, leading to a visually distracting user experience.

---

## ✅ Solution Options

### Option 1: Remove Repeating Updates

If the data does not need to update live:

```javascript
// Call only once
updateData();

// REMOVE this line
// setInterval(updateData, 5000);
```

---

### Option 2: Disable Animation

If updates are needed but animation should be suppressed, add this to the chart config:

```javascript
options: {
    animation: false,
    responsive: true,
    scales: {
        y: {
            beginAtZero: true
        }
    }
}
```

---

### Option 3: Update Chart Only If Data Has Changed

To allow auto-refresh without unnecessary animation:

```javascript
let lastData = [];

function updateData() {
    fetch('/dashboard/presence')
        .then(response => response.json())
        .then((output) => {
            // Only update chart if data has changed
            if (JSON.stringify(output) !== JSON.stringify(lastData)) {
                myBar.data.datasets[0].data = output;
                myBar.update();
                lastData = output;
            }
        });
}
```

---

## 🧠 Recommendation

Use a combination of:

* ✅ `animation: false` to reduce UI flickering.
* ✅ Conditional chart updates using `JSON.stringify()` to avoid unnecessary redraws.
* ❌ Avoid using `setInterval` unless real-time changes are expected.

---

## 🔁 Summary

| Aspect         | Before                       | After Fix                      |
| -------------- | ---------------------------- | ------------------------------ |
| Chart Behavior | Reanimates every 5 seconds   | Only updates on data change    |
| Animation      | Forced, repeated animation   | Disabled or conditional        |
| UX Impact      | Distracting, flickering bars | Smooth and optimized rendering |

---

Let Astralis light the unknown ⚡
