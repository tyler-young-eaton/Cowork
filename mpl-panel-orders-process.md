# MPL Daily Sales-Report Review — Panels & Data Quality

> **Owner:** Tyler D. Young (Sr. Design Engineer, Eaton MPL)
> **Maintained in git** — see the Change Log at the bottom. Copilot proposes updates and supplies a commit note; Tyler commits.
> **Last updated:** 2026-07-09 (v1.4)

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

## 9. Building a Panel/Substation BOM from Configurator XML

Turn an Eaton Bid Manager takeoff export into the electrical BOM that loads to the AML. Applies to **Marina Panels, Event Panels, and Substations**.

### 9a. Inputs
- **Panel takeoff XML** — `PNLXS*.xml` (PRL4X), per-panel `*-MPLW*.xml` (PRL1X), or interior takeoffs `*-<neg>*.xml` (event/marina pedestals, the `…I` interior item). One file per panel/designation. **Authoritative** for catalog numbers.
- **Substation summary (MD)** — `*-SEBC*.md` etc.: enclosure, transformer, GFM, primary breaker, panel. Use for substation-level items not in the panel XML. When XML and MD disagree, the **latest ALT / XML wins**.
- **Stamp on every row:** GO#, Job, Line Item (= panel designation/item, e.g. `001` / `001A` / `001I`). Column **G** = the negotiation/ALT reference (e.g. `MPLW0702X5K1 - 0010`, `PA720417X6K3 - 000`).

### 9b. Output columns (match `Active_AML_Log`)
`GO | Line Item | Job | Part No. | Part Descr. | Qty | Details/Supplier/Drop Ship Address (G)`

### 9c. Categories to output
Breakers · Shunt trips · CTs · RCMS/GFM · SPDs · **Transformer** (substations) · **Panel/Panelboard** (one per designation). Exclude chassis, bus, ground bar, covers, nameplates, seismic labels, **provisions/spaces** (`TWIN_SPACE`, `PROVBAB1`), and **standard receptacles** (see 9f).

### 9d. Special mods (`BM_SpecMod`) — read these first
Special mods appear in the XML as `Class="BM_SpecMod"` nodes (also logged as `AddMod … SpecificationNotes` in the `ChangeLog`). They drive two things:
- **Catalog-prefix flip.** The takeoff `Pnl_Catnum` (`PUN…`) is the *with-special-mods* catalog. MPL orders the panel with the mods **stripped**, which flips the prefix — seen so far **`PUN → PAS`** (event panel) and **`PUN → PBS`** (substation). So the AML panel catalog ≠ the raw `Pnl_Catnum`. If the stripped prefix isn't certain, use the raw catnum and **flag it**.
- **Ground-fault counts.** GFM mods ("Multi Circuit Ground Fault Monitor – N Circuits") give the number of ground-fault circuits, which sets the shunt / CT / GFM counts (see 9g).

### 9e. Breakers — part numbers & who orders the main
| Family | Rule |
|---|---|
| **PD / PDD / PDG** | Base `BuildCatnum_cache` with the trailing `N…N` stripped **+ `J`** (standard lugs). e.g. `PDG43G0800TFANNNNNNN` → **`PDG43G0800TFAJ`**; `PDD23F0100TFFNNNNNNN` → **`PDD23F0100TFFJ`**. Mains **and** branches. |
| **FD** | Standard catalog `FD<poles><amps>` (e.g. `FD2100`). Used when a **2-pole PD can't take a shunt** — convert PD→FD by matching **amps + poles**. |
| **BAB (PRL1X)** | Standard catalog as-is (`BAB2100S`, `BAB2050`, `BAB1020`). The **`S` suffix = integrated shunt trip** — no separate shunt line. |
| **Others** (QBH, GHQ, NGS, QB…GF) | Standard ordering catalog. |

**Main-breaker handling — depends on panel type:**
- **Substations:** the purchased panel ships **Main Lug Only (MLO)** — the main is removed and side-mounted (frees pole space), so the **main IS a separate order line** (e.g. Anchor Cove `PDD32G0300TFAJ`; TCM 800A mains).
- **Marina & Event panels:** the main **stays on the purchased panel** and arrives with it → **no separate main line** (unless noted). (North Coast Montana `PDD23F0100`; Northville Downs.)

### 9f. Panel / panelboard line
- **Part No.** = the (mods-stripped) **catalog number**; **Descr** = **Product ID + designation** (e.g. `Pow-R-Line1X Wall Mount Event Panel`; `Pow-R-Line4X Marina Unit Substation – SB1`). One line per designation; consolidate identical catnums.
- **Receptacles / duplex / cordsets left off** — standard stocked items unless the order specifies otherwise. Enclosure / box / trim handled separately if tracked at all.

### 9g. Ground-fault triad (shunt + CT + GFM) & the E-stop exception
- **Every ground-fault-protected circuit = three parts:** a **shunt trip** on the breaker, a **CT**, and a channel on a **ground-fault monitor**. So **# shunts = # CTs = # ground-fault circuits** (from the GFM mods) — they must reconcile.
- **Shunts:** FD → `SNT1RP08K`; PD frame-matched `PDG3XST130ACDCS` (F3) / `PDG4XST130ACDCS` (F4), coil `130ACDC`; BAB integrated (`…S`, no line). Match shunt frame to breaker frame. **Never output `CN…` placeholders.**
- **CTs — one per ground-fault circuit, by amps:** 15–100A `C311CT35` · 125–250A `C311CT60` · 300–600A `CTAC120` · 700–1200A `CTAC210`. Descr = "NN mm Current Transformer".
- **GFM:** multi-circuit `RCMS490D-A2` (≤12 circuits each; units = one per "N Circuits" mod, i.e. ceil(circuits ÷ 12)); single-circuit `RCM420-D-2`.
- **⚠ E-stop exception:** a breaker (usually a main) with a **shunt but no CT** is an **E-stop**, not ground fault — **no CT, no GFM channel**. E-stops aren't always labeled; read them from **shunt-without-CT + the main / panel-config context** and **call them out**.

### 9h. Transformer (substations) — from the **MPL Eaton Transformer List**
Match **KVA + phase + primary/secondary voltage + windings + rise**. **Phase tell:** `120/240V` secondary = **1-phase**; `120/208V` = **3-phase**. Worked: 75KVA 1Ph 480→120/240 Cu 150C = **`T20P11S7516CCCU`** (not the 3Ph `V48…7516`, which is 120/208).

### 9i. Consolidation & quantities
Group identical Part No. across designations onto **one** row; Line Item lists them all (`001,002,003` / `001I,002I,003I,004I`); sum Qty. Don't multiply by repeated layout instances.

### Worked examples
- **TCM Substations (`EUX0003724`)** — PRL4X substation: PD mains (base+J) + FD-converted 2-pole branches + `SNT1RP08K`; the two 800A main shunts have **no CT = E-stops**; branch shunt+CT+GFM triad; `RCMS490D-A2` ×4 (one per GFM note); panel catnum flipped `PUN → PBS`.
- **Anchor Cove Marina (`SSE1514103`)** — 3× PRL1X marina unit **substations** (MLO → mains ordered): `PDD32G0300TFAJ` main + BAB branches (`BAB2100S` integrated shunt); `T20P11S7516CCCU` 75KVA transformer; ground-fault triad = `BAB2100S` + `C311CT35` + `RCMS490D-A2` (2 circuits/sub); `PDG32F0150TFAJ` primary breaker.
- **North Coast Montana (`STA1503274`)** — 4× EP-W-6 wall-mount **event** panels: panel `PASAEABTB18N` (`PUN → PAS` after stripping the `CN3908936` event-panel mod), Descr "Pow-R-Line1X Wall Mount Event Panel" ×4; `BAB1020` ×16 / `BAB1030` ×4 / `BAB2050` ×4; **no main line** (100A `PDD23F0100` stays on the panel); **no shunts/CTs/GFM** (no ground fault); receptacles off.

---

## 10. Dropping a BOM into the Master AML

### 10a. Column layout (`Active_AML_Log`, header on row 2)
`A` GO · `B` Line Item · `C` Job · `D` Part No. · `E` Part Descr. · `F` Qty · `G` Details/Supplier/Drop Ship Address · `H`+ buyer / order# / dates / received (procurement) · `AA` AML Shelf Check.

### 10b. Substation entry pattern (mirror existing subs)
Itemize: **transformer** + **panelboard per designation** + **every breaker** + **shunts** + **RCMS/CTs**, consolidating identical parts across designations. Cross-checked against St. Andrews, Puerto Los Cabos, Norfolk, Naples, Harbor View.

### 10c. Placement rules (writing to the shared AML)
1. **Only write empty cells; never change existing data.**
2. **Keep the whole BOM contiguous** — find the next group of adjacent empty rows big enough for all rows; don't scatter into the first stray blanks.
3. **⚠ Find the TRUE data end.** Column **AA ("AML Shelf Check")** is pre-filled with **"No"** far below the last real entry, so "last non-empty row" is misleading. Use the **last row that has data in A–G**, and drop the BOM in the rows immediately below it.
4. **Write only A–G.** Leave H–Z for procurement; leave the pre-existing AA "No" untouched.
5. **Targeted cell-range write** (Excel range API), not a file re-upload, so only the target cells change.
6. **Show the exact target rows + values and get approval before writing.**

### Worked example
- `SSE1514103` (10 rows) — real A–G data ended at **row 658** (the TCM job), though the AA "No" ran to 3498. Placed in **A659:G668**.
- `STA1503274` (4 rows) — appended at **A669:G672**; re-pulled the workbook first (it had changed since the prior read) before confirming the data end.

---

## Change Log

| Date | Version | Change |
|---|---|---|
| 2026-07-08 | 1.0 | Initial document — data sources, panel-requiring product codes, reconciliation process, caveats, scheduled task, and 07-08-2026 snapshot. |
| 2026-07-08 | 1.1 | Clarified each run reads the **current date's** report (or the most recent available); reconciliation works from the **Backlog** tab; relabeled §7 as an example snapshot. |
| 2026-07-09 | 1.2 | Broadened the task to **panels + data-quality scan**; added catalog-signature detection (§3), the data-quality scan (§7), and the four-tab styled output workbook (§6); renamed the task; refreshed the snapshot to 07-09-2026. |
| 2026-07-09 | 1.3 | Added **§9 XML→BOM build rules** (PD base+`J` ordering; PD→FD 2-pole-shunt conversion; BAB integrated-shunt family; frame-matched shunts; CT sizing; RCMS/GFM; transformer lookup incl. 1Ph vs 3Ph tell; panelboard line; cross-designation consolidation) and **§10 Master AML drop-in procedure** (substation entry pattern; write only empty A–G; keep the BOM contiguous; find the true data end past the pre-filled "No" in col AA). Worked examples: TCM Substations (PRL4X PD/FD) and Anchor Cove Marina `SSE1514103` (PRL1X BAB, placed at A659:G668). |
| 2026-07-09 | 1.4 | Reworked **§9** from the event-panel job: **§9d special-mod detection** (`BM_SpecMod` nodes → catalog-prefix flip `PUN→PAS/PBS`, and ground-fault counts); **main-breaker rule** (substations = MLO → main ordered; marina/event panels = main stays on panel → no line); **§9f panel line** = catalog # + Product ID + designation, receptacles off; **§9g ground-fault triad** (shunts = CTs = GFM circuits) with the **E-stop exception** (shunt without CT). Added the North Coast Montana `STA1503274` event-panel worked example (placed at A669:G672). |
