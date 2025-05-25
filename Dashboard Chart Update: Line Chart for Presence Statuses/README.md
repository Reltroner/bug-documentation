# 📈 Dashboard Chart Update: Line Chart for Presence Statuses

This update enhances the dashboard's presence visualization by:

- Replacing the existing **bar chart** with a **multi-line chart**.
- Displaying separate trends for each status:
  - ✅ Present
  - ❌ Absent
  - 🕒 Late
  - 📝 Leave

---

## 🐞 Previous Problem

The original chart:

- Displayed only **`present`** data.
- Used a **bar chart** which was not ideal for trend analysis.
- Lacked comparison between different presence statuses over time.

---

## ✅ New Behavior

A responsive **Chart.js Line Chart** now shows:

- 4 lines, one for each `status`.
- Monthly trend data from January to December.
- Real-time summary of attendance types.

---

## 🔧 Backend Changes

**File:** `app/Http/Controllers/DashboardController.php`

**New method:**
```php
public function presence()
{
    $statuses = ['present', 'absent', 'late', 'leave'];
    $response = [];

    foreach ($statuses as $status) {
        $rawData = Presence::where('status', $status)
            ->selectRaw('MONTH(date) as month, COUNT(*) as total')
            ->groupBy('month')
            ->orderBy('month')
            ->get();

        $data = array_fill(0, 12, 0);
        foreach ($rawData as $item) {
            $data[$item->month - 1] = $item->total;
        }

        $response[$status] = $data;
    }

    return response()->json($response);
}
````

---

## 🎨 Frontend Changes

**Chart Type:** `line`
**Library:** Chart.js (`chart.umd.js`)

**New script:**

```javascript
const presenceChart = new Chart(ctxLine, {
    type: 'line',
    data: {
        labels: ["Jan", "Feb", ..., "Dec"],
        datasets: []
    },
    options: {
        responsive: true,
        tension: 0.3,
        animation: false,
        plugins: {
            legend: { position: 'top' },
            title: { display: true, text: 'Latest Presences by Status' }
        },
        scales: {
            y: { beginAtZero: true, stepSize: 1 }
        }
    }
});

function updateData() {
    fetch('/dashboard/presence')
        .then(res => res.json())
        .then(data => {
            presenceChart.data.datasets = [
                { label: 'Present', data: data.present, borderColor: '#4caf50' },
                { label: 'Absent',  data: data.absent,  borderColor: '#f44336' },
                { label: 'Late',    data: data.late,    borderColor: '#ff9800' },
                { label: 'Leave',   data: data.leave,   borderColor: '#2196f3' }
            ];
            presenceChart.update();
        });
}

updateData();
```

---

## 🧠 Benefit

| Feature           | Before        | After         |
| ----------------- | ------------- | ------------- |
| Chart Type        | Bar Chart     | Line Chart    |
| Status Categories | 1 (`present`) | 4 (all)       |
| UX Clarity        | Basic         | Trend-aware   |
| Trend Visibility  | Hard to spot  | Clearly shown |

---

Let Astralis light the unknown ⚡

