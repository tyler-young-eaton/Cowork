# Outlook Inbox — Copilot Handling Rules

> **Owner:** Tyler D. Young (Sr. Design Engineer, Eaton MPL)
> **Maintained in git** — see the Change Log at the bottom. Copilot proposes updates and supplies a commit note; Tyler commits.
> **Last updated:** 2026-07-09 (v1.2)

---

## 1. Purpose

A living rule set for how Copilot triages Tyler's Outlook inbox and handles email-driven tasks — so cleanup stays consistent, nothing assigned to him slips through, and the inbox stays near-zero.

---

## 2. Triage Basics

- **Do NOT archive.** Mark emails **read** in place.
- **Delete** only confirmed spam.
- **Default end state:** anything that isn't a live task assigned to Tyler ends up **read**; his actual tasks stay surfaced (Section 3) and **flagged** (Section 4).

---

## 3. Tasks Assigned to Tyler (top priority)

- When an email **specifically tasks Tyler** — something to **do**, or something he must **read / review** — keep it **surfaced at the front of context** (a running, visible list) so he can recall it quickly.
- Do **not** silently mark these read; handle per Section 4 (flag + track).
- **Tracking:** log actionable tasks in the **“Cowork Tasks” Planner plan** — plan ID `bmXBL3QcYkuqjBgFgvhEomQABW-g`, **“To do”** bucket — assigned to Tyler. **Do NOT use the “Project Brief” plan.** Planner tasks assigned to Tyler **sync into To Do**.

---

## 4. Category Handling

| Category | Action | Notes |
|---|---|---|
| **Emails that specifically task Tyler** — something to **do** or to **read / review** | **Flag** the email + due date the **following week** | Covers Mfg. Hold List (Microsoft Lists) mentions **and MOC sign-offs Tyler must read** (he reviews them; he does **not** sign). Flag tool can't set the date — add a **Cowork Tasks** Planner task dated next week as the date-carrying fallback. Keep surfaced (Section 3). |
| **Compliance / EatonUniversity training** | Mark read | Tyler batches these on a set day (e.g., Friday) |
| **Digests / newsletters / Viva Engage / promos** | Mark read | |
| **Teams / Loop / Lists notifications**, meeting-acceptance notices | Mark read | |
| **Shift pass-downs** | Mark read | |
| **Monitor-only threads** (Tyler CC'd, nothing tasked to him) | Mark read | e.g., ATS Enclosure, 47-68400 louver plates, PPAP, 167KVA transformer |
| **Meeting invites** | Present for **accept / decline** | Don't auto-decide unless told to just mark read |

---

## 5. Communication Style

- **Teams messages to Tyler himself = brief notification only** (e.g., `👋 Hi Tyler — Copilot here. [X] is complete.`). Keep the detail in the Copilot chat, not in Teams.

---

## 6. Housekeeping / Open Options

- Consider an **auto-read rule** for “Eaton Daily/Weekly Digest” to stop the pileup. *(Not yet set up.)*

---

## Change Log

| Date | Version | Change |
|---|---|---|
| 2026-07-08 | 1.0 | Initial rule set — triage basics (mark-read-not-archive), assigned-task tracking, category handling (Mfg Hold List flag + due date, MOC = management sign-off, compliance batched, digests/monitor → read), brief Teams pings, and housekeeping. |
| 2026-07-09 | 1.1 | Task tracking now targets the **“Cowork Tasks”** Planner plan (not “Project Brief”); Mfg Hold List date-fallback tasks also go to Cowork Tasks. |
| 2026-07-09 | 1.2 | Generalized the flag rule: **flag any email that specifically tasks Tyler with something to do or to read/review** (not just Mfg. Hold List). Clarified **MOC sign-offs** — Tyler still **reads** them (flag), he just isn't the signer. Switched doc to git-maintained (Tyler commits). |
