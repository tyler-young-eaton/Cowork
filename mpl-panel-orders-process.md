# MPL Daily Panel-Orders → Master AML Process

> **Owner:** Tyler D. Young (Sr. Design Engineer, Eaton MPL)
> **Maintained in git** — see the Change Log at the bottom. Copilot proposes updates and supplies a commit note; Tyler commits.
> **Last updated:** 2026-07-08 (v1.1)

---

## 1. Purpose

Each business day, identify the open marine orders that **require an electrical panel to be ordered/built**, and reconcile them against the **Master AML** procurement log so nothing that needs parts placed slips through. The output is a short list of orders that are **not yet on the AML** (i.e. still need action), plus a supporting spreadsheet.

---

## 2. Data Sources

### 2a. Daily Order & Sales Report (source of open orders)
- **Location:** MPL-Operations SharePoint → `Shared Documents` → `General` → **`Sales-Orders Reports`**
- **Folder logic:** `Sales-Orders Reports / {YYYY} / {MM-MonthName} / {MM-DD-YYYY}.xlsm`
  - Example: `2026 / 07-July / 07-08-2026.xlsm`
- **Naming:** `MM-DD-YYYY.xlsm` (macro-enabled). The file dated a given day contains data **through the prior working day** (the "Yesterday" column).
- **Key tab used:** **`Backlog`** — every open order line. Relevant columns:
  `General Order Number`, `General Order Item`, `Job Name`, `Cust Name`, `Catalog Number`,
  `Item Quantity Open`, `Item Status`, `Required Ship Date`, `Commit Ship Date`,
  `Product Code`, `Product Code Description`, `Product Line`, `Item Extended Amount`.
- Other useful tabs: `Daily Orders` (bookings), `Invoice Data` (shipments), `Product Codes` (code→description→line map), `Order Overview Pivot`.

### 2b. Master AML (source of "what's been placed")
- **Location:** MPL-Operations SharePoint → `Shared Documents` → `General` → **`Master AML.xlsx`**
- **Key tab:** **`Active_AML_Log`** (~3,500 rows; header on **row 2**). A procurement log keyed by **`GO` (General Order number)** + **`Job`**, listing the parts/breakers to be purchased per order, with buyer, `Order #`, ship date, and received date.
- Archive/other tabs: `Completed and Archived AML Log`, `Archived_AML_2025/2024/2023`, `Excess Inventory`.
- **Match key between the two files: the General Order number** (`General Order Number` in the report = `GO` in the AML).

---

## 3. Panel-Requiring Product Categories

Orders in these product codes require a panel to be ordered:

| Category | Product Code | Catalog description | Product Line | Notes |
|---|---|---|---|---|
| **Marina Panels** | `BC34A` | Panels | Marine | Catalog family `PUN…` |
| **Substations** | `BC34M` | Substation - Marine | Marine | |
| **Substations (sub-assembly)** | `BC34C` | Sub Assembly - Substations | Marine | Currently 0 open |
| **Event Panels** | `BC28` | Portable Power | Marine | Catalog prefix `EP-` |
| **Mega Yachts** | `BC33` | Mega Yachts | Marine | Currently 0 open |

---

## 4. Process (per run)

1. Open the daily report for the **current date** (Section 2a folder logic). **If there is no file for today** — weekend, holiday, or simply not posted yet — use the **next most recent available** file instead. Never read a fixed/older date.
2. Work from the **`Backlog`** tab only — that is all the reconciliation needs. (Only fall back to a full-workbook search if it genuinely helps your analysis.) In `Backlog`, filter rows where **`Product Code`** is one of `BC34A`, `BC34M`, `BC34C`, `BC28`, `BC33`.
3. Collect the key columns (Section 2a) for each matching line.
4. For each order's **General Order number**, check whether it appears in **`Master AML.xlsx → Active_AML_Log`**.
   - **Present** → already placed on the AML.
   - **Absent** → **needs to be placed** (action item).
5. Compare against the **previous business day's** report to flag changes: new orders, orders that dropped off (shipped/closed), and status changes (e.g. newly on Hold).
6. Produce a spreadsheet: **`Panel Orders to Verify vs AML - MM-DD-YYYY.xlsx`** with two tabs:
   - **Panel Orders** — every matching line item, incl. an `On Active AML?` column.
   - **Summary by Order** — one row per General Order, with lines, open value, AML status, and notes.

---

## 5. Caveats / Gotchas

- **AML match is GO#-level only.** A "YES" means the order *number* appears in the log — **not** that every required panel part for that order has been entered. Still verify part-by-part before closing an order out.
- **Mis-coded orders slip past the filter.** An order can require a panel but be booked under a non-panel code.
  - *Known example:* **`STA1510549`** (Gebhart Electric / Dock Boxes Unlimited Sarasota) is booked under **Steel / Nema 4X Boxes (`P3200`)**, but item `001I` catalog `PUNCBFBTB24E` is from the `PUN…` pedestal/panel family, and its `001B / 001I / 001T` suffixes read as **Box / Internal / Trim** of a pedestal. Orders like this won't be caught by the product-code filter — spot-check `PUN…`-family catalog items sitting under Steel/Nema codes.
- **Item status codes:** `O` = Open (in process), `H` = Hold; a separate `T hold` tab lists T-status items. Full definitions live on the report's `Item Status Code` tab.

---

## 6. Scheduled Task

| Field | Value |
|---|---|
| **Name** | Daily Panel-Orders vs AML Check |
| **Schedule** | Every day, **7:00 AM America/New_York (Eastern)** |
| **Run mode** | Fresh/stateless each run (re-derives from that day's files) |
| **What it does** | Opens the report for **the current date** (or the **most recent available** file if today's isn't posted) → reads the **`Backlog`** tab (full-workbook search only if needed) → pulls the panel-requiring open orders (Section 3) → compares to the Active AML by GO# → flags day-over-day changes → highlights orders still needing AML placement |
| **Delivery** | Brief **Teams message to Tyler**; detailed line-item spreadsheet saved to his files |

*Open options (not yet changed): weekdays-only instead of 7-day; email instead of Teams.*

---

## 7. Example Snapshot — 07-08-2026 report (data through 07-07-2026)

> Point-in-time example captured when this doc was written. **Each run reads the current date's report (or the most recent available file), not this fixed date** — the numbers below will differ day to day.

- **76 open line items** across **13 General Orders**, **$1,255,433** total open value.
- By category: Marina Panels **46 / $337,328** · Substations **29 / $915,058** · Event Panels **1 / $3,047** · Mega Yachts **0**.
- **Orders NOT yet on the Active AML (action items):**
  - `SSE1514103` — Anchor Cove Marina (Substation) — ~$56,863
  - `STA1503274` — North Coast Montana (Marina Panels) — ~$9,604
  - `SML1408670` — Oyster Point Marina — $3 (appears in the 2025 archive; likely a leftover line)

---

## Change Log

| Date | Version | Change |
|---|---|---|
| 2026-07-08 | 1.0 | Initial document — data sources, panel-requiring product codes (BC34A/BC34M/BC34C/BC28/BC33), reconciliation process, caveats, scheduled task, and 07-08-2026 snapshot. |
| 2026-07-08 | 1.1 | Clarified that each run reads the **current date's** report (or the most recent available file when today's is missing) rather than a fixed date; noted the reconciliation works from the **Backlog** tab only, with full-workbook search optional; relabeled §7 as an example snapshot. |
