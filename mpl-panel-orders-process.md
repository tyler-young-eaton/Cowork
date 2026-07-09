# MPL Daily Sales-Report Review — Panels & Data Quality

> **Owner:** Tyler D. Young (Sr. Design Engineer, Eaton MPL)
> **Maintained in git** — see the Change Log at the bottom. Copilot proposes updates and supplies a commit note; Tyler commits.
> **Last updated:** 2026-07-09 (v1.2)

---

## 1. Purpose

Each business day, review the newest daily Order & Sales Report and produce one workbook that does two jobs:
1. **Panel-order → AML reconciliation** — find the open marine orders that **require an electrical panel to be ordered/built** and check each against the **Master AML** procurement log, so nothing that needs parts placed slips through.
2. **Data-quality scan** — flag potential errors anywhere in the report (mislabels, bad dates, blanks, etc.) for review.

---

## 2. Data Sources

### 2a. Daily Order & Sales Report (source of open orders)
- **Location:** MPL-Operations SharePoint → `Shared Documents` → `General` → **`Sales-Orders Reports`**
- **Folder logic:** `Sales-Orders Reports / {YYYY} / {MM-MonthName} / {MM-DD-YYYY}.xlsm`
  - Example: `2026 / 07-July / 07-09-2026.xlsm`
- **Naming:** `MM-DD-YYYY.xlsm` (macro-enabled). The file dated a given day contains data **through the prior working day**. (The month folder also holds daily `.csv` exports of the Backlog/Orders/Sales.)
- **Key tab used:** **`Backlog`** — every open order line. Relevant columns:
  `General Order Number`, `General Order Item`, `Job Name`, `Cust Name`, `Catalog Number`,
  `Item Quantity Open`, `Item Status`, `Required Ship Date`, `Commit Ship Date`,
  `Product Code`, `Product Code Description`, `Product Line`, `Item Extended Amount`.

### 2b. Master AML (source of "what's been placed")
- **Location:** MPL-Operations SharePoint → `Shared Documents` → `General` → **`Master AML.xlsx`**
- **Key tab:** **`Active_AML_Log`** (~3,500 rows; header on **row 2**). A procurement log keyed by **`GO` (General Order number)** + **`Job`**, listing the parts/breakers to be purchased per order.
- **Match key between the two files: the General Order number** (`General Order Number` in the report = `GO` in the AML).

---

## 3. Panel-Requiring Product Categories

Orders in these product codes require a panel to be ordered:

| Category | Product Code | Catalog description | Product Line | Notes |
|---|---|---|---|---|
| **Marina Panels** | `BC34A` | Panels | Marine | Catalog family `PUN…` |
| **Substations** | `BC34M` | Substation - Marine | Marine | |
| **Substations (sub-assembly)** | `BC34C` | Sub Assembly - Substations | Marine | |
| **Event Panels** | `BC28` | Portable Power | Marine | Catalog prefix `EP-` |
| **Mega Yachts** | `BC33` | Mega Yachts | Marine | |

**Detection uses product code AND catalog signature.** A panel can be booked under a non-panel code (see §5), so also treat any line whose **Catalog Number** is from the pedestal-panel family (`PUN…`) as panel content, regardless of its product code.

---

## 4. Process (per run)

1. Open the daily report for the **current date** (Section 2a folder logic). **If there is no file for today** — weekend, holiday, or not yet posted — use the **next most recent available** file. Never read a fixed/older date.
2. Work from the **`Backlog`** tab (full-workbook review only if it helps). Identify panel-requiring lines by **product code** (`BC34A/BC34M/BC34C/BC28/BC33`) **and** by **catalog signature** (`PUN…`).
3. For each order's **General Order number**, check the **`Master AML → Active_AML_Log`**: **Logged** if present, **MISSING** if absent (= needs to be placed).
4. Mark **New Today** for orders that appear for the first time versus the previous business day's report.
5. Run the **data-quality scan** (see §7) across the whole report.
6. Produce the styled workbook (see §6) and send the Teams summary.

---

## 5. Caveats / Gotchas

- **AML match is GO#-level only.** "Logged" means the order *number* appears in the log — **not** that every required panel part has been entered. Verify part-by-part before closing an order out.
- **Mis-coded orders** — an order can require a panel but be booked under a non-panel code. The catalog-signature check (§3) now catches these.
  - *Known example:* **`STA1510549`** (Gebhart Electric) is booked under **Steel / Nema 4X Boxes (`P3200`)**, but item `001I` catalog `PUNCBFBTB24E` is a `PUN…` pedestal, and its `001B / 001I / 001T` suffixes read as **Box / Internal / Trim**. Caught by catalog, missed by code.
- **Item status codes:** `O` = Open, `H` = Hold, `T` = T-hold. Full definitions live on the report's `Item Status Code` tab.

---

## 6. Scheduled Task & Output

| Field | Value |
|---|---|
| **Name** | Daily Sales-Report Review — Panels & Data Quality |
| **Schedule** | Every day, **7:00 AM America/New_York (Eastern)** |
| **Run mode** | Fresh/stateless each run |
| **What it does** | Opens the current (or most recent) report → panel reconciliation vs the Active AML (§3–4) → data-quality scan (§7) → produces the workbook below → sends a Teams summary |
| **Delivery** | Concise **Teams message to Tyler** (MISSING orders + data-quality counts); full workbook saved to his files, named with the report date |

**Output workbook — four tabs:**
1. **Overview** — panel counts (total / Logged / MISSING / New Today) + data-quality tally by type.
2. **AML Reconciliation Summary** — one row per order (AML Status, New Today, Customer, Job, Product Code(s), Panel Type, # Panel Lines, Total Qty Open, Backlog Value, Items on Hold, Required Ship) + a **TOTAL** row.
3. **Panel Line Items** — every panel line, so each order traces to its exact lines.
4. **Data-Quality Flags** — every flagged line with Issue Type + Detail.

**Formatting:** navy title banners, blue headers, thin gray borders, **red MISSING / green Logged / gold New Today** cells, a light-blue **TOTAL** row, and a color **legend**. Save under a date-stamped filename each day (avoids same-name caching).

*Open options (not changed): weekdays-only instead of 7-day; email instead of Teams; tighten the data-quality scan (drop holds / narrow zero-price) if the flag volume is too noisy.*

---

## 7. Data-Quality Scan

Flags — **for review only, nothing is changed** — across all product families:

| Issue Type | What it catches |
|---|---|
| **Catalog/Code mismatch** | A catalog whose family implies a different product code than assigned (the `STA1510549` class) |
| **Unmapped catalog (#N/A)** | Catalog not found in the product-code map |
| **Blank/zero open qty** | Open quantity missing or zero |
| **Zero/blank unit price** | Price missing or zero |
| **Implausible ship date** | Required/commit dates far in the future (e.g. 2028–2029), a commit date before the order date, or malformed values |
| **On Hold / T-hold** | Items in `H` or `T` status (high-volume; candidate to drop if noisy) |

---

## 8. Example Snapshot — 07-09-2026 report

> Point-in-time example. **Each run reads the current date's report** — numbers differ day to day.

- **Panels:** 15 orders (**10 Logged**, **5 MISSING**), 85 line items.
- **MISSING from the Active AML (action items):** `LTA0041698` (Sarasota Yacht Club), `SSE1514103` (Anchor Cove Marina), `STA1503274` (North Coast Montana), `STA1510549` (Gebhart Electric — the mislabeled one), `SML1408670` (Oyster Point).
- **Data-quality:** 313 flags — Catalog/Code mismatch 12 · Unmapped #N/A 6 · Blank qty 10 · Zero price 59 · Implausible date 28 · Hold/T-hold 198.

---

## Change Log

| Date | Version | Change |
|---|---|---|
| 2026-07-08 | 1.0 | Initial document — data sources, panel-requiring product codes, reconciliation process, caveats, scheduled task, and 07-08-2026 snapshot. |
| 2026-07-08 | 1.1 | Clarified each run reads the **current date's** report (or the most recent available); reconciliation works from the **Backlog** tab; relabeled §7 as an example snapshot. |
| 2026-07-09 | 1.2 | Broadened the task to **panels + data-quality scan**; added catalog-signature detection (§3), the data-quality scan (§7), and the four-tab styled output workbook (§6); renamed the task; refreshed the snapshot to 07-09-2026. |
