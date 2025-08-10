# 📌 Bug Report: `chk_transactions_balanced` Constraint Violation During Seeding

### **Overview**

When running:

```bash
php artisan migrate:fresh --seed
```

the seeding process fails with the following error:

```
SQLSTATE[HY000]: General error: 3819 Check constraint 'chk_transactions_balanced' is violated.
```

This occurs during the `TransactionSeeder` execution, specifically when creating and updating transaction header totals (`total_debit`, `total_credit`, `total_debit_base`, `total_credit_base`).

---

### **Root Cause**

The `chk_transactions_balanced` database constraint enforces:

```
total_debit = total_credit
AND
total_debit_base = total_credit_base
```

In the original `TransactionFactory::withLines()` implementation:

* Random detail lines are generated first.
* The "flip last row" logic was intended to ensure balance, **but it failed when the number of lines was odd or when rounding differences occurred**.
* This resulted in the transaction header being updated with **unbalanced totals**, which immediately triggered the database constraint error.

---

### **Example Failing Case**

**Target totals from header:**

```
total_debit  = 6792.64
total_credit = 6792.64
```

**Generated detail lines (simplified):**

| Line | Debit   | Credit     |
| ---- | ------- | ---------- |
| 1    | 3500.00 | 0          |
| 2    | 0       | 1200.00    |
| 3    | 3292.64 | 0          |
| 4    | 0       | 24036.88 ❌ |

> The last credit value was incorrectly flipped without recalculating the exact balancing amount.

**Result:**
Header totals become:

```
total_debit  = 6792.64
total_credit = 24036.88
```

→ Fails `chk_transactions_balanced` constraint.

---

### **Solution**

Replace the "flip last row" balancing approach with a **two-phase balancing method**:

1. Generate `n - 2` random lines.
2. Use the last **two lines** to perfectly match the remaining debit and credit amounts.
3. Add a **safety auto-balancing line** if rounding differences occur (optional but recommended).

---

### **Patched Implementation**

The fixed method ensures header totals are updated **only after** details are balanced.

See [updated `withLines()` implementation](./database/factories/TransactionFactory.php) for details.

---

### **Status**

✅ **Fixed** — Seeding now completes successfully without violating `chk_transactions_balanced`.
