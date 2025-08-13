# Floating‑Point Balance Check Bug in Transaction Edit Form

> This README documents a UI bug where the **Update Transaction** button remains disabled even when totals look balanced. It affects the ERP Finance journal forms that calculate debit/credit totals in the browser.

---

## TL;DR

* **Symptom:** Banner shows `Diff: -0.00` and the **Update Transaction** button stays disabled.
* **Root cause:** JavaScript floating‑point precision (e.g., `3516.06 - 3516.06` becomes `-0.0000000001`).
* **Fix:** Compare totals using a small tolerance (**epsilon**) or round to 2 decimals before comparison.

---

## Affected Screens

* `resources/views/transactions/edit.blade.php`
* (Likely) `transactions/create.blade.php`, `transaction-details/create.blade.php`, and `transaction-details/edit.blade.php` if they use the same `diff === 0` guard.

---

## How to Reproduce

1. Open **Edit Transaction** for any entry with multiple lines.
2. Adjust debits/credits until both totals display the same value (e.g., `3516.06` vs `3516.06`).
3. Observe the banner text shows `Diff: -0.00` and the **Update Transaction** button is still disabled.

---

## Why It Happens

JavaScript uses binary floating‑point arithmetic. Sums that look equal at two decimals may differ by a tiny amount at machine precision (e.g., `-0.0000000001`).
The UI previously checked `diff === 0`, which fails under these tiny discrepancies.

---

## The Fix (Front‑End)

Round to 2 decimals for display and use an **epsilon** tolerance (e.g., 0.005) for the balance check.

Replace the `recalc()` function in `resources/views/transactions/edit.blade.php` with the version below (apply the same pattern to other forms using the guard):

```js
function recalc() {
  let d = 0, c = 0, lines = 0;

  tableBody.querySelectorAll('tr').forEach(tr => {
    const dv = parseFloat(tr.querySelector('.details-debit')?.value || 0);
    const cr = parseFloat(tr.querySelector('.details-credit')?.value || 0);
    if (dv > 0 || cr > 0) lines++;
    d += dv; c += cr;
  });

  // Round to 2 decimals for totals
  const d2 = Math.round(d * 100) / 100;
  const c2 = Math.round(c * 100) / 100;

  const rate = parseFloat(exRateEl?.value || 1) || 1;
  const dBase = Math.round(d2 * rate * 100) / 100;
  const cBase = Math.round(c2 * rate * 100) / 100;

  sumDebitEl.textContent   = d2.toFixed(2);
  sumCreditEl.textContent  = c2.toFixed(2);
  sumExrateEl.textContent  = rate;
  sumDebitBEl.textContent  = dBase.toFixed(2);
  sumCreditBEl.textContent = cBase.toFixed(2);

  // Use epsilon (0.5 cent tolerance)
  const EPS = 0.005;
  const diff = d2 - c2;
  const isBalanced = Math.abs(diff) < EPS && d2 > 0 && lines >= 2;

  balanceDeltaEl.textContent = isBalanced
    ? '(Balanced)'
    : `(Diff: ${diff.toFixed(2)} | Lines: ${lines})`;
  balanceDeltaEl.className = isBalanced ? 'text-success' : 'text-danger';

  submitBtn.disabled = !isBalanced;
}
```

### Notes

* Keep the **one‑sided** per‑row rule (either debit **or** credit) as is; this fix only targets the floating‑point comparison.
* The **server‑side** validation already uses `round(..., 2)` and ensures strict balance, so aligning the UI with an epsilon makes the UX consistent without weakening server rules.

---

## Validation Checklist

* [ ] With totals showing the same 2‑decimal value, the banner shows **Balanced** and the button is **enabled**.
* [ ] Introduce a genuine mismatch (e.g., add `0.01` on one side): banner shows `Diff: 0.01` and the button is **disabled**.
* [ ] Exchange rate changes update base totals correctly and do not reintroduce the bug.
* [ ] Repeat on mobile and desktop breakpoints.

---

## Related Improvements (Optional)

* Extract the `recalc()` logic into a shared script and reuse across `create/edit` pages for Transactions and Transaction Details.
* When posting forms, rely on server validation for the final authority; the client‑side guard is a UX helper only.

---

## Status

**Fixed (UI)** by applying epsilon/rounding in `recalc()`.
**Server** logic unchanged and remains the source of truth.
