# Outlook Inbox — Copilot Handling Rules

> **Owner:** Tyler D. Young (Sr. Design Engineer, Eaton MPL)
> **Maintained collaboratively** — see the Change Log at the bottom. Copilot proposes updates; Tyler approves.
> **Last updated:** 2026-07-08 (v1.0)

---

## 1. Purpose

A living rule set for how Copilot triages Tyler's Outlook inbox and handles email-driven tasks — so cleanup stays consistent, nothing assigned to him slips through, and the inbox stays near-zero.

---

## 2. Triage Basics

- **Do NOT archive.** Mark emails **read** in place.
- **Delete** only confirmed spam.
- **Default end state:** anything that isn't a live task assigned to Tyler ends up **read**; his actual tasks stay surfaced (Section 3).

---

## 3. Tasks Assigned to Tyler (top priority)

- When an email **specifically tasks Tyler**, keep it **surfaced at the front of context** (a running, visible list) so he can recall it quickly.
- Once the task is noted, the notification email may be marked **read**.
- **Tracking:** log actionable tasks in **Planner → “My Tasks”** (assigned to Tyler). Microsoft To Do isn't directly writable, but Planner tasks assigned to him **sync into To Do**.

---

## 4. Category Handling

| Category | Action | Notes |
|---|---|---|
| **Mfg. Hold List mentions** (Microsoft Lists) | **Flag** the email + due date the **following week** | Flag tool can't set the date — use a Planner task dated next week as the date-carrying fallback |
| **MOC sign-off** emails | Mark read | For **management** to sign, not Tyler |
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
