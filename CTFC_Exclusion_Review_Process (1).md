# CTFC Exclusion List Review Process

---

## Overview

Every week, a scheduled search pulls all active KV store exclusions expiring within 30 days and caches them with risk hit data. The engineer on rotation reviews each entry, sets a determination, and submits the week to a Strategist for approval. Once approved, the engineer applies the changes to the KV store.

This document covers how to access the dashboard, how to complete each role's steps, and what to do when things go wrong.

---

## Accessing the Dashboard

Navigate to the CTFC Exclusion List Review dashboard in Splunk Cloud.

**Direct URL:** `https://es-wf-sp.splunkcloud.com/en-US/app/wf-ctfc-content/CTFC_Exclusion_List_Review_Tracker`

**If the URL doesn't work:**
1. Go to `https://es-wf-sp.splunkcloud.com`
2. Select **Apps > WF | CTFC - Content**
3. Find **CTFC Exclusion List Review** in the navigation bar at the top of the app

**If it's not in the nav bar:**
1. Select **Settings > User Interface > Views**
2. Filter by name: `CTFC_Exclusion`
3. Select **Open** in the Actions row

The dashboard loads with the KPI panels and week info visible immediately. Nothing else appears until you select a role.

---

## How the Dashboard Works

A saved search called `exclusion_bulk_risk_check` runs every Monday at 00:00 UTC. It pulls all active entries expiring within 30 days, checks `index=risk` for 30-day and 90-day hit counts, and caches the results. The dashboard reads from that cache on load. No live queries run against the KV store or risk index.

**If the dashboard shows no entries:**
1. Go to **Activity > Search Jobs**, set filter to **All Users**
2. Look for a job with `exclusion_bulk_risk_check` in the name
3. If it shows **Done** with a non-zero result count, the cache is populated -- click Refresh
4. If the job is not listed, the search has not run yet -- contact the engineer on rotation to trigger it manually
5. If it shows **Done** with zero results, either nothing is expiring this cycle (verify in the KV store before closing the ticket) or the search failed silently -- if active entries are expiring and the count is still zero, contact the engineer on rotation

---

## Known Behavior: Risky Command Prompt

Every write action in this dashboard triggers a Splunk security warning about risky commands. This is a platform-level policy applied to any search that writes data and cannot be disabled from the dashboard.

**Every write follows this pattern:**
1. Select **Confirm** from the dropdown
2. Splunk shows a "risky command" dialog
3. Click **Run Anyway**
4. Select **Confirm** from the dropdown again

Nothing is lost or duplicated by confirming twice. The first confirmation is consumed by the popup.

> A request to exempt this dashboard from this prompt is pending: CTFCSE-8470 Splunk | Risky command exemption request - outputlookup - wf-ctfc-content

---

## Traffic Flag Values

| Flag | What it means | What to do |
|---|---|---|
| ACTIVE | The IoC is hitting risk events right now | Recommend extend |
| Validate in Notables / Jira | Zero risk index hits in 90 days | Check Mission Control notables and the Jira ticket before deciding |

> "Validate in Notables / Jira" does not mean the entry is inactive. It means the risk index had no hits for that specific rule and IoC combination. The entry could still be suppressing activity elsewhere. Verify before disabling anything.

---

## Engineer Workflow

### Step 1: Start of Rotation

Before opening the dashboard, confirm a Jira ticket exists for your rotation week.

- Naming convention: **Exclusion List Review YYYY-Www** (the ISO week shown on the dashboard, e.g. Exclusion List Review 2026-W13)
- If one has not been assigned to you, create it through the Engineer Intake / Submit Ideas process
- If you are taking over from the previous engineer, review their ticket for carry-over items, pending owner responses, or anything flagged for follow-up

### Step 2: Open the Dashboard

Set the **Role** dropdown to **Engineer**. The pending entries table and Action dropdown appear.

### Step 3: Review Entries

The table shows all entries expiring within 7 days. Each row contains:

| Column | What it means |
|---|---|
| IoC Value | The suppressed value in the KV store |
| Content | The correlation rule this entry is tied to |
| Expires | When the entry expires |
| Days Left | Days until expiry |
| Traffic | ACTIVE or Validate in Notables / Jira |
| 30d Hits | Times the IoC appeared in risk events in the last 30 days |
| 90d Hits | Times the IoC appeared in risk events in the last 90 days |
| Last Seen | Most recent date the IoC appeared in risk events |
| Owner | Team that owns the correlation rule |

**How to decide on each entry:**

1. **Traffic = ACTIVE** -- extend it. It is actively suppressing events.
2. **Traffic = Validate in Notables / Jira** -- check Mission Control notables for the rule, open the Jira ticket on the entry, and check the CTFC procedure guide for the rule. If the notable is still active and relevant, extend. If the notable is closed and there has been no activity in 90 days, disable.
3. **Owner is not CTFC** -- contact the owning team before deciding. Get their determination and write their response in your notes before submitting.

> CTFC does not own notables. The team that created the rule owns them. If SCD is listed as the owner, check the SCD ticket on the entry to find the actual owning team.

### Step 4: Set Determinations

Choose your method from the **Action** dropdown.

**Bulk Review** -- use when multiple entries under the same rule all get the same determination:
1. Type part of the rule name into the **Content filter** (or leave * for all entries)
2. From the **Apply Filter** dropdown, select **Apply Filter** -- this reveals the Determination and Notes fields
3. Select a determination from the **Determination** dropdown
4. Enter notes in the **Notes** field (ticket number, reason, or N/A)
5. Select **Confirm** from the **Confirm Bulk Submit** dropdown
6. If the risky command prompt appears, click **Run Anyway**, then select **Confirm** again

**Single Entry Review** -- use when an entry needs its own determination or different notes:
1. Click any row in the pending table -- the entry details populate in the form fields above
2. Select a determination from the **Determination** dropdown
3. Enter notes in the **Notes** field
4. Select **Confirm** from the **Confirm Entry Submit** dropdown
5. If the risky command prompt appears, click **Run Anyway**, then select **Confirm** again

**Determination options:**

| Option | When to use it |
|---|---|
| Extend - 1 year | Active or still needed. No reason to re-review soon. |
| Extend - 6 months | Active but worth checking again sooner. |
| Extend - 3 months | Borderline. Short extension while you gather more information. |
| Disable - no activity | No hits, no active notables, no reason to keep it. |
| Disable - no longer needed | The original issue was resolved or the entry was temporary. |
| Modify - scope change needed | The rule, IoC value, or scope needs updating before extending. |

Notes are required for everything except Extend - 1 year. Strat will see your notes.

### Step 5: Submit the Week to Strat

Once every entry has a determination, switch the **Action** dropdown to **Submit Week to Strat**. The table updates to show your determinations for the week.

1. Confirm every entry in the table looks correct before proceeding
2. Enter your Jira ticket number (e.g. CTFCSE-1234) in the **Jira Ticket Number** field
3. Select **Confirm** from the **Confirm Submit to Strat** dropdown
4. If the risky command prompt appears, click **Run Anyway**, then select **Confirm** again

The week moves to pending Strat approval. The Jira ticket number is written to every entry for that week.

---

## Strategist Workflow

Open the same dashboard and set the **Role** dropdown to **Strategist (Strat)**.

1. Click the row for your week in the **Weeks awaiting your decision** table
2. The full list of entries loads below with the engineer's determinations, notes, traffic data, and Jira ticket
3. Review the entries -- if anything needs clarification, check the Jira ticket or contact the engineer before deciding
4. Select **Approved** or **Denied** from the **Decision** dropdown
5. Enter notes in the **Notes** field (required if denying -- be specific about what needs to change)
6. Select **Confirm** from the **Confirm Decision** dropdown
7. If the risky command prompt appears, click **Run Anyway**, then select **Confirm** again

**Approved:** The Jira ticket moves toward closure via automation. The engineer applies the approved entries to the KV store using the SCD Exclusion List Input dashboard once the ticket closes.

**Denied:** The week returns to the engineer's pending queue marked for re-review. The engineer updates the affected determinations and resubmits. Your notes are the only guidance they have -- be specific.

---

## After Approval: Applying to the KV Store

1. Open the **SCD Exclusion List Input** dashboard
2. For each approved entry, apply the updated expiry or disable as determined
3. Confirm each change

Once all entries are applied, close the Jira ticket.

> This step is manual during initial rollout and will be evaluated for automation after the first full cycle.

---

## End of Rotation: Handoff

1. Create a new Jira ticket for the incoming engineer: **Exclusion List Review YYYY-Www** (next ISO week)
2. Assign it to the incoming engineer
3. Add notes for anything carrying over -- pending owner responses, modified entries, anything unusual from this cycle

Your original ticket closes at this point. The new ticket carries the context forward.

---

## Decision History

The **Decision History** panel at the bottom of the dashboard shows every past Strat decision -- approved and denied weeks, entry counts, who decided, when, and any notes. Visible to all roles at all times.

**To query history outside the dashboard:**

```spl
| inputlookup exclusion_review_tracker.csv
| stats
    count as total_entries,
    count(eval(determination!="" AND determination!="Needs follow-up")) as reviewed,
    count(eval(strat_status="approved")) as approved,
    count(eval(strat_status="denied")) as denied,
    count(eval(strat_status="pending_strat")) as pending_strat,
    values(jira_ticket) as jira_tickets,
    values(strat_updated_by) as strat_reviewer
    by iso_week
| sort - iso_week
```
