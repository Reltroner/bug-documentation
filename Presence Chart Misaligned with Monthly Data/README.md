# 🛠 Fix: Presence Chart Misaligned with Monthly Data

This fix addresses a visual and logical bug in the dashboard's "Latest Presences" bar chart, where the displayed data did not align correctly with the month labels.

---

## 🐞 Problem

The presence chart pulls attendance data using the following controller logic:

```php
$data = Presence::where('status', 'present')
    ->selectRaw('MONTH(date) as month, COUNT(*) as total_present')
    ->groupBy('month')
    ->orderBy('month', 'asc')
    ->get();

$temp = [];
$i = 0;
foreach ($data as $item) {
    $temp[$i] = $item->total_present;
    $i++;
}
````

### ❌ Issues:

* The `labels` array is fixed: `["Jan", "Feb", ..., "Dec"]`.
* But the `data` array only contains entries for **months that have data**, in sequential order.
* This **misaligns bars with labels**, leading to a misleading chart.

---

## ✅ Solution

Refactor the controller to explicitly assign data to the correct month indexes (0–11):

```php
public function presence()
{
    $rawData = Presence::where('status', 'present')
        ->selectRaw('MONTH(date) as month, COUNT(*) as total_present')
        ->groupBy('month')
        ->orderBy('month', 'asc')
        ->get();

    // Initialize 12 months with 0
    $data = array_fill(0, 12, 0);

    foreach ($rawData as $item) {
        $monthIndex = $item->month - 1;
        $data[$monthIndex] = $item->total_present;
    }

    return response()->json($data);
}
```

---

## ✅ Results

* The chart now always has 12 bars (January to December).
* If a month has no data, the chart will display a value of `0`.
* The bars are now correctly aligned with their respective months.

---

## 📊 Before vs After

| Month | Raw Data Exists? | Chart Without Fix | Chart With Fix |
| ----- | ---------------- | ----------------- | -------------- |
| Jan   | ✅ Yes            | ✅                 | ✅              |
| Feb   | ❌ No             | ❌ Shifted         | ✅ Shown as 0   |
| Mar   | ✅ Yes            | ✅                 | ✅              |
| Apr   | ✅ Yes            | ✅                 | ✅              |
| ...   | ...              | ...               | ...            |

---

## 🔁 Summary

**Bug Type**: Data alignment and visualization inconsistency
**Root Cause**: Ignoring empty months when building dataset
**Fix Type**: Normalize data array to 12 elements, map by month index

---

Let Astralis light the unknown ⚡
