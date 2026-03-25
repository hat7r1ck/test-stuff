# CTFC Exclusion List Review - Deployment Package

**Date:** 2026-03-25  
**Author:** Jeffrey Morris  
**Scope:** Dashboard updates, procedure documentation

This file contains everything needed to push the latest round of changes to the CTFC Exclusion List Review system. Two sections:

1. **Procedure Documentation** (updated, simplified)
2. **Dashboard XML Snippets** (header buttons + zero-week logging)

---
---

# Part 1: Procedure Documentation

> Copy this into the Confluence page. Replaces the current content of **CTFC.Splunk Exclusion List Review Process**.

---

# CTFC.Splunk Exclusion List Review Process

**Space:** CTFC Documentation  
**Scope:** Splunk Cloud (WF02958.KV.IOC_Whitelist)  
**Last Updated:** March 2026

---

## Purpose

Step-by-step instructions for CTFC Tier II engineers and Strategists to perform the weekly review of expiring IOC whitelist entries in Splunk Cloud.

---

## Overview

The WF02958.KV.IOC_Whitelist (KV store) is a list of IOCs maintained by the CTFC. Each entry suppresses alert creation for predefined non-malicious activity, preventing XSOAR case generation for the CTFC. Suppression may be temporary or permanent depending on the IOC and its justification. Periodic review is required to confirm each entry remains valid.

The CTFC Exclusion List Review is a weekly process for evaluating active KV store suppression entries before they expire. The engineer on rotation reviews each entry against risk data and notable data, assigns a determination, and submits the week to a Strategist for approval. Once approved, the engineer applies the updated entries to the KV store.

---

## Requirements

Each reviewed entry in WF02958.KV.IOC_Whitelist must have the following fields populated upon completion:

- Submitted date
- Expiry date within one year of the submitted date
- Notes containing an XSOAR case number, a Jira ticket reference (CPSS/SCR/CTFCSE), or detailed justification if no ticket reference exists
- Submitter AD-ENT username

Additional guidelines:

- One year of inactivity is the threshold for recommending an entry be disabled
- Expiry dates for all entries must not be set to the same date, as this will cause a flood of reviews in a future week
- VAT scanning IPs are excluded from the review scope and are reviewed through a separate process via the CTFC Service Desk

> **Rotation handoff:** CTFC Tier II engineers rotate weekly. When your rotation ends, create a Jira ticket for the next engineer using the naming convention below. This replaces the old handover meeting process.

> **Notable ownership:** Not all entries in the exclusion list are CTFC-owned. Always review the ticket referenced in the entry to identify the owning team before making a determination.

---

## Step 1: Start of Rotation

- Confirm a Jira ticket exists for your rotation week
  - **Naming convention:** `Exclusion List Review YYYY-Www` (e.g. Exclusion List Review 2026-W13)
  - The ISO week number is shown on the dashboard once loaded
- If a ticket has not been assigned to you, create one through the Engineer Intake / Submit Ideas [CTFCSE Intake Portal](JIRA_INTAKE_URL)
- If you are taking over from the previous engineer, review their handoff ticket for carry-over items, pending owner responses, or anything flagged for follow-up before starting your review

---

## Step 2: Access the Dashboard

Navigate to the dashboard:

- **Direct Link:** [https://es-wf-sp.splunkcloud.com/en-US/app/wf-ctfc-content/CTFC_Exclusion_List_Review_Tracker](https://es-wf-sp.splunkcloud.com/en-US/app/wf-ctfc-content/CTFC_Exclusion_List_Review_Tracker)

If the direct link does not work:

1. Select **Settings** > **User Interface** > **Views**
2. Filter by name "CTFC_Exclusion_List_Review_Tracker"
3. Select **Open** located in the Actions row

Once loaded, the **Current Week**, **Data Last Refreshed**, and **KPI panels** are visible.

Set the **Role** dropdown to **Engineer**. The pending entries table and Action dropdown will appear.

- Check the **Data Last Refreshed** timestamp aligns to the Monday for your rotation week
  - A saved search called `exclusion_bulk_risk_check` runs every Monday at Midnight UTC (00:00)
  - This cron job runs the query to pull all active entries expiring within 30 days, checks `index=risk` for hit counts (30-day and 90-day), finds the last time each IoC appeared in risk events, and caches the results
  - If the date shows the Monday for your rotation week, then there is nothing expiring this week
  - If the date shows a past date, then the cache is stale
    - To check job status: Go to **Activity** > **Jobs** > **All apps** > search `exclusion_bulk_risk_check`

---

## Step 3: Pick a Review Method

Use the **Action** dropdown to choose how you want to work:

| Action | When to use it |
|---|---|
| **Bulk Review** | Multiple entries under the same rule all get the same determination |
| **Single Entry Review** | An entry needs its own determination or different notes |
| **Submit Week to Strat** | You are done reviewing and ready to hand off for approval |

---

## Step 4: Review Entries

The dashboard shows all entries expiring within 7 days. Each row contains:

| Column | Description |
|---|---|
| IOC Value | The suppressed value |
| Content | The correlation rule associated with this entry |
| Expires | Date the entry expires |
| Traffic | ACTIVE or Validate in Notables / Jira |
| 30d Hits | Times the IOC appeared in risk events in the last 30 days |
| 90d Hits | Times the IOC appeared in risk events in the last 90 days |
| Last Seen | Most recent date the IOC was seen in risk events |
| Owner | Team that owns the correlation rule |

### Traffic Flag Values

| Flag | What it tells you | What to do |
|---|---|---|
| **ACTIVE** | The IOC hit risk events within the last 90 days | Recommend extend |
| **Validate in Notables / Jira** | Zero matching risk events in 90 days | Check Mission Control notables and the Jira ticket before deciding |

"Validate in Notables / Jira" does not mean the entry is inactive. It means the risk index had no hits for that specific rule and IOC combination. The entry could still be doing work elsewhere. Verify before you disable anything.

### How to decide

1. **Traffic = ACTIVE:** It is suppressing events right now. Extend it.
2. **Traffic = Validate in Notables / Jira with 0 hits across both windows:**
   - Go to Mission Control and look at the notables for this rule
   - Open the Jira ticket on the entry
   - Open the CTFC procedure guide for the rule to see if there is any traffic
   - Notable still active and relevant: extend
   - Notable closed and no activity in 90 days: disable
3. **Owner is not CTFC:** Contact the owning team. Get their determination. Write their response in your notes before submitting.

> CTFC does not own notables. The team that created the rule owns them. If SCD is listed as the owner, check the SCD ticket on the entry to find the actual owning team.

---

## Step 5: Submit Determinations

### Bulk Review

1. Type part of the rule name into the **Content filter** (or leave `*` for everything)
2. Select **Apply Filter** to show the write fields
3. Pick a determination from the dropdown
4. Add your notes (ticket number, reason, or N/A)
5. Select **Confirm** from the **Confirm Bulk Submit** dropdown
6. Risky command prompt appears. Click **Run Anyway**, then select **Confirm** again

### Single Entry Review

1. Click any row in the review table
2. The entry details populate in the fields above the table
3. Pick a determination and add notes
4. Select **Confirm** from the **Confirm Entry Submit** dropdown
5. Risky command prompt appears. Click **Run Anyway**, then select **Confirm** again

### Determination Options

| Option | When to use it |
|---|---|
| Extend - 1 year | Active or still needed. No reason to re-review soon |
| Extend - 6 months | Active but worth checking again sooner |
| Extend - 3 months | Borderline. Short extension while you gather more information |
| Disable - no activity | No hits, no active notables, no reason to keep it |
| Disable - no longer needed | The original issue was resolved or the entry was temporary |
| Modify - scope change needed | The rule association, IOC value, or scope needs updating before extending |

Notes are required for everything except Extend - 1 year. Strat will see your notes.

---

## Step 6: Submit the Week to Strat

Once every entry for the week has a determination, switch the **Action** dropdown to **Submit Week to Strat**.

Before submitting, create (if not already done) a Jira ticket through the [CTFCSE Intake Portal](JIRA_INTAKE_URL). Include:

- Every rule that is up for expiry
- Your recommendation for each (extend or disable)
- Your rationale (traffic data, Jira history, owner follow-up, case count)

Then:

1. Review the determinations table. Confirm every entry is correct before proceeding
2. Enter your Jira ticket number (e.g. CTFCSE-1234) in the **Jira Ticket Number** field
3. Select **Confirm** from the **Confirm Submit to Strat** dropdown
4. If the risky command prompt appears, click **Run Anyway**, then select **Confirm** again

The week moves to pending Strat approval. The Jira ticket number is written to every entry for that week and will appear in the Strat queue.

---

## Splunk Cloud: Strategist Workflow

Open the same dashboard and set the **Role** dropdown to **Strategist (Strat)**.

1. In the **Weeks awaiting your decision** table, click the row for the week you are reviewing
2. All entries load below with the engineer's determinations, notes, traffic data, and Jira ticket number
3. Review the entries. If anything requires clarification, check the Jira ticket or contact the engineer directly before submitting your decision
4. Select **Approved** or **Denied** from the **Decision** dropdown
5. Enter notes in the **Notes** field (required if denying. Be specific about what needs to change)
6. Select **Confirm** from the **Confirm Decision** dropdown
7. If the risky command prompt appears, click **Run Anyway**, then select **Confirm** again

**Approved:** The Jira automation advances the ticket. The engineer applies the approved entries to the KV store once the ticket closes.

**Denied:** The week goes back to the engineer with your notes. The engineer addresses the issues, updates determinations as needed, and resubmits. Your notes are the primary guidance. Be specific.

### Decision History

The Decision History panel at the bottom of the dashboard shows every past Strat decision. Approved and denied weeks are both shown with the entry count, who decided, when, and any notes. This panel is visible to all roles and updates automatically.

---

## After Approval: Applying Entries to the KV Store

> **Do not use the Lookup Editor app to edit or delete entries.** The Lookup Editor has caused most of the expiry issues that already exist. Entries must never be deleted. They serve as the historical record.

Once the Jira ticket has closed via automation:

1. Open the [SCD Exclusion List Input](https://es-wf-sp.splunkcloud.com/en-US/app/wf-ctfc-content/SCD_Exclusion_List_Input) dashboard
2. Apply each approved entry. Update the expiry for extensions, disable entries marked for removal
3. Confirm each change

> This step is manual during initial rollout and will be evaluated for automation after the first full cycle.

**[TODO: Link to SCD Exclusion List Input procedure document]**

### Verify the Write

After updating an entry, confirm it was written correctly:

```spl
| inputlookup WF02958.KV.IOC_Whitelist where value="REPLACE_WITH_IOC_VALUE"
| search content=*
| eval expiry=expiry/1000
| eval submitted_date=submitted_date/1000
| fieldformat expiry = strftime(expiry, "%Y-%m-%d %H:%M")
| fieldformat submitted_date = strftime(submitted_date, "%Y-%m-%d %H:%M")
| sort -content
```

Swap `REPLACE_WITH_IOC_VALUE` for the actual IOC value.

---

## End of Rotation: Handoff

1. Create a new Jira ticket for the incoming engineer: `Exclusion List Review YYYY-Www` (next ISO week)
2. Assign it to the incoming engineer
3. Document any carry-over items in the ticket. Pending owner responses, modified entries, or anything that requires follow-up in the next cycle

Your original ticket closes at this point. The new ticket carries the context forward.

---

## Jira Ticket Naming Convention

This process uses the ISO 8601 (ISO 8601-1:2019) week date standard to eliminate ambiguity.

> **Footnote:** [Introduction to the new ISO 8601-1 and ISO 8601-2](https://www.iso.org/cms/render/live/en/sites/isoorg/contents/news/2019/02/Ref2366.html)

ISO 8601 assigns every week in the year a number from W01 through W52/W53. Weeks always start on Monday and end on Sunday. Week 01 is defined as the week containing the first Thursday of the year. This is the same standard Splunk and the dashboard uses for `iso_week` in the review tracker.

> **Footnote:** [Splunk - Date and time format variables](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference/Commontimeformatvariables)

**New naming convention:**

```
Exclusion List Review YYYY-WXX
```

Examples:
- ~~Exclusion List Review MM_YYYY_Week X~~ (old)
- `Exclusion List Review 2026-W11`
- `Exclusion List Review 2026-W12`

The year and week number come directly from the ISO week shown on the dashboard. The scope of each ticket is exactly the entries displayed for that ISO week on the dashboard.

**When your rotation ends:**

1. Create a new Jira ticket using the naming convention above for the next ISO week
2. Assign it to the incoming engineer
3. Add any notes about open items, pending owner responses, or entries that need follow-up from the current week

---

## How the Dashboard Works

A saved search called `exclusion_bulk_risk_check` runs every Monday at Midnight UTC (00:00). It pulls all active entries expiring within 30 days, checks `index=risk` for hit counts (30-day and 90-day), finds the last time each IOC appeared in risk events, and caches the results.

When you open the dashboard, it loads from that cache. No live queries run against the KV store or risk index on load.

**If the dashboard is empty**, the saved search may not have completed yet:

1. Go to **Activity** > **Jobs** > **All apps**
2. Search for `exclusion_bulk_risk_check`
3. Confirm the status shows **Done** and the result count is not zero

---

## Known Behavior: Risky Command Prompt

Splunk Cloud shows a security warning every time a search writes to a CSV lookup using `outputlookup`. This is a platform-level policy that applies to any search writing data and cannot be turned off from the dashboard side.

**What happens on every write:**

1. You select **Confirm** from the dropdown
2. Splunk shows a "risky command" dialog
3. Click **Run Anyway**
4. Select **Confirm** from the dropdown a second time (the first selection was consumed by the popup)

Every write action in this dashboard follows that pattern: Confirm, Run Anyway, Confirm again. Nothing is lost or duplicated by confirming twice.

> **Info:** A request to exempt this dashboard from the prompt has been submitted to the platform team. Until that goes through, this is the expected behavior.
> CTFCSE-8470 Splunk | Risky command exemption request - outputlookup - wf-ctfc-content

---

## Backup: Review History Query

To check the status of past weeks outside the dashboard:

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
| rename iso_week AS "Week",
         total_entries AS "Total Entries",
         reviewed AS "Reviewed",
         approved AS "Approved",
         denied AS "Denied",
         pending_strat AS "Pending Strat",
         jira_tickets AS "Jira Tickets",
         strat_reviewer AS "Strat Reviewer"
```

One row per week. Use it to confirm a week was fully processed or to audit prior weeks.

---

## Setting Up a Confluence Review Calendar

Use a Confluence Team Calendar to track which engineer is responsible for which review week.

**Create the calendar:**

1. Open the CTFC Documentation space in Confluence
2. Click **Calendars** in the space sidebar
3. Click **+** > **Add new calendar**
4. Name it "CTFC Exclusion Review Rotation"
5. Click **Create**

**Enable week numbers (admin required):**

1. Go to **General Configuration** > **Team Calendars** > **Global Settings**
2. Check **Display week numbers**
3. Click **Save**

**Add rotation events:**

1. Click a date on the calendar
2. Set the event name to the engineer (e.g. "Jeffrey Morris - Exclusion Review")
3. Set the date range to Monday through Friday
4. Toggle recurring if the rotation is fixed
5. Click **Save**

**Embed on a Confluence page:**

1. Edit the target page
2. Type `/team calendar` and select the macro
3. Pick the calendar, set the view to Month or Timeline
4. Publish

---

## Links

- [CTFC Exclusion List Review Dashboard](https://es-wf-sp.splunkcloud.com/en-US/app/wf-ctfc-content/CTFC_Exclusion_List_Review_Tracker)
- [SCD Exclusion List Input](https://es-wf-sp.splunkcloud.com/en-US/app/wf-ctfc-content/SCD_Exclusion_List_Input)
- [Content Management](https://es-wf-sp.splunkcloud.com/en-US/app/wf-ctfc-content/content_management)
- Team share: `\\NCCCNSF701Z1.wellsfargo.net\c_cis_groups\CTFC\CTFC_Security_Wiki\Attachments\Splunk_Exclusion_List_Review_Process`

---

## Splunk On-Premises

This process covers Splunk Cloud only. The on-premises review process has not changed. Refer to the existing Confluence documentation for the WF02958.KV.IOC_Whitelist Validation Spreadsheet process using the CTFC Incident Management Metrics dashboard.

> **Warning | Splunk Expiry:** CHECK EXCLUSION ITEMS FOR BOTH SPLUNK CLOUD AND SPLUNK ON-PREM

---
---

# Part 2: Dashboard XML Snippets

> These are copy-paste blocks for the live dashboard. The dashboard has diverged from the workspace copy, so use these snippets instead of replacing the full XML.

---

## 2A. Header Buttons Update

Replace the existing `ROW 0: HEADER` HTML panel content with this:

```xml
<html>
  <div style="padding:12px 16px; background-color:#1a1a1a; border-left:4px solid #006d9c; border-radius:4px; overflow:hidden;">
    <span style="float:right; margin-left:16px; display:flex; gap:8px; align-items:center;">
      <a href="CONFLUENCE_DOC_URL"
         style="background-color:#444; color:#ffffff; padding:4px 12px; border-radius:3px; text-decoration:none; font-size:0.85em; font-weight:bold;">
        Docs
      </a>
      <a href="JIRA_INTAKE_URL"
         style="background-color:#444; color:#ffffff; padding:4px 12px; border-radius:3px; text-decoration:none; font-size:0.85em; font-weight:bold;">
        Jira Intake
      </a>
      <a href="https://es-wf-sp.splunkcloud.com/en-US/app/wf-ctfc-content/SCD_Exclusion_List_Input"
         style="background-color:#f8be34; color:#1a1a1a; padding:4px 12px; border-radius:3px; text-decoration:none; font-size:0.85em; font-weight:bold;">
        SCD KV Input
      </a>
      <a href="https://es-wf-sp.splunkcloud.com/en-US/app/wf-ctfc-content/CTFC_Exclusion_List_Review_Tracker"
         style="background-color:#006d9c; color:#ffffff; padding:4px 12px; border-radius:3px; text-decoration:none; font-size:0.85em; font-weight:bold;">
        Refresh
      </a>
    </span>
    <p style="color:#cccccc; margin:0; font-size:0.9em;">
      <strong style="color:#ffffff;">Engineers:</strong> Select role, pick action. For Bulk: type filter, Apply Filter, set determination, then Confirm.
      <span style="margin-left:24px;"></span>
      <strong style="color:#f8be34;">Strat:</strong> Select Strategist, click a row in the Awaiting Approval table, then select Confirm dropdown to submit your decision.
    </p>
  </div>
</html>
```

**Placeholders to replace:**
- `CONFLUENCE_DOC_URL` - URL of the Confluence procedure page
- `JIRA_INTAKE_URL` - URL of the CTFCSE Intake Portal

---

## 2B. Zero-Entry Week Logging

Allows engineers to log a week that had zero expiring entries so the decision history has a complete record.

### Snippet 1: Add to `<init>` block (before `</init>`)

```xml
    <!-- Zero-entry week -->
    <unset token="show_zero_week"></unset>
    <unset token="zero_week_eligible"></unset>
    <unset token="zero_week_fire"></unset>
```

### Snippet 2: Background searches (after `</fieldset>`, before `<search id="base_expiring">`)

```xml
  <search id="zero_week_check">
    <query>| makeresults
| eval iso_week=strftime(now(), "%G-W%V")
| join type=left iso_week
    [| inputlookup exclusion_review_tracker.csv
     | where iso_week=strftime(now(), "%G-W%V")
     | stats count as already_logged by iso_week]
| eval already_logged=coalesce(already_logged, 0)
| where already_logged=0</query>
    <earliest>-1m</earliest>
    <latest>now</latest>
    <done>
      <condition match="$job.resultCount$ &gt; 0">
        <set token="zero_week_eligible">1</set>
      </condition>
      <condition match="$job.resultCount$ == 0">
        <unset token="zero_week_eligible"></unset>
      </condition>
    </done>
  </search>

  <search id="zero_week_gate">
    <query>| loadjob savedsearch="jeffrey.morris@wellsfargo.com:wf-ctfc-content:exclusion_bulk_risk_check"
| stats count as expiring_count
| where expiring_count=0</query>
    <earliest>-1m</earliest>
    <latest>now</latest>
    <done>
      <condition match="$job.resultCount$ &gt; 0">
        <set token="show_zero_week">1</set>
      </condition>
      <condition match="$job.resultCount$ == 0">
        <unset token="show_zero_week"></unset>
      </condition>
    </done>
  </search>
```

### Snippet 3: UI panels (after KPI row, before pending table row)

Only shows when: 0 expiring entries AND nothing logged this week AND engineer role selected.

```xml
  <row depends="$show_zero_week$,$zero_week_eligible$,$show_engineer$">
    <panel>
      <html>
        <div style="padding:12px 18px; background-color:#1a2a1a; border-left:6px solid #53a051; border-radius:4px; overflow:hidden;">
          <span style="color:#53a051; font-size:1.05em; font-weight:bold;">No entries expiring this week.</span>
          <span style="color:#cccccc; font-size:0.9em; margin-left:16px;">Use the dropdown below to log this week as having zero entries. This creates a record in the tracker so the decision history shows this week was checked.</span>
        </div>
      </html>
    </panel>
  </row>
  <row depends="$show_zero_week$,$zero_week_eligible$,$show_engineer$">
    <panel>
      <input type="dropdown" token="zero_week_confirm">
        <label>Log Zero-Entry Week</label>
        <choice value="">Select to log</choice>
        <choice value="go">Confirm</choice>
        <default></default>
        <initialValue></initialValue>
        <change>
          <condition value="go">
            <set token="zero_week_fire">1</set>
            <unset token="zero_week_confirm"></unset>
            <unset token="form.zero_week_confirm"></unset>
          </condition>
          <condition>
            <unset token="zero_week_fire"></unset>
          </condition>
        </change>
      </input>
    </panel>
  </row>
  <row>
    <panel depends="$zero_week_fire$">
      <title>Zero-Entry Week Log Result</title>
      <table>
        <search>
          <query>| makeresults
| eval ioc_value="N/A",
       content="No entries expiring this week",
       determination="No review needed",
       review_notes="Zero entries in scope for this review period",
       reviewed_by="$env:user$",
       review_date=strftime(now(), "%Y-%m-%d"),
       review_epoch=now(),
       iso_week=strftime(now(), "%G-W%V"),
       original_expires="N/A",
       wf_rule_owner="N/A",
       traffic_flag="N/A",
       hit_count_30d="0",
       hit_count_90d="0",
       last_seen_display="N/A",
       jira_ticket="",
       strat_status="no_review_needed",
       strat_updated_by="$env:user$",
       strat_update_date=strftime(now(), "%Y-%m-%d"),
       strat_notes="Zero entries in scope",
       fkey=sha256("zero-week|".iso_week."|".tostring(review_epoch))
| fields ioc_value, content, determination, review_notes, reviewed_by, review_date, review_epoch, iso_week, original_expires, wf_rule_owner, traffic_flag, hit_count_30d, hit_count_90d, last_seen_display, jira_ticket, strat_status, fkey, strat_updated_by, strat_update_date, strat_notes
| eval _write_gate="$zero_week_fire$"
| where _write_gate="1"
| append
    [| inputlookup exclusion_review_tracker.csv]
| fields - _*
| fields ioc_value, content, determination, review_notes, reviewed_by, review_date, review_epoch, iso_week, original_expires, wf_rule_owner, traffic_flag, hit_count_30d, hit_count_90d, last_seen_display, jira_ticket, strat_status, fkey, strat_updated_by, strat_update_date, strat_notes
| outputlookup exclusion_review_tracker.csv override_if_empty=false
| where ioc_value="N/A" AND iso_week=strftime(now(), "%G-W%V")
| eval result="Week " . iso_week . " logged with zero entries"
| table result</query>
          <earliest>-1m</earliest>
          <latest>now</latest>
          <done>
            <condition match="$job.resultCount$ &gt; 0">
              <unset token="zero_week_fire"></unset>
              <unset token="show_zero_week"></unset>
              <unset token="zero_week_eligible"></unset>
              <eval token="refresh_trigger">now()</eval>
            </condition>
            <condition match="$job.resultCount$ == 0">
              <unset token="zero_week_fire"></unset>
            </condition>
          </done>
        </search>
        <option name="drilldown">none</option>
        <option name="refresh.display">progressbar</option>
      </table>
    </panel>
  </row>
```

### Snippet 4: Decision History updates

In the decision history panel, update the `status_display` eval. Replace the existing `case()` with:

```spl
| eval status_display=case(strat_status=="approved", "APPROVED", strat_status=="denied", "DENIED", strat_status=="pending_strat", "AWAITING STRAT", strat_status=="no_review_needed", "NO ENTRIES", 1==1, strat_status)
```

Update the colorPalette on the Status field:

```xml
<colorPalette type="map">{"APPROVED":"#53a051","DENIED":"#dc4e41","AWAITING STRAT":"#f8be34","NO ENTRIES":"#006d9c"}</colorPalette>
```

---
---

# Changelog

| Date | Change |
|---|---|
| 2026-03-25 | Added header buttons (Docs, Jira Intake, SCD KV Input, Refresh) |
| 2026-03-25 | Added zero-entry week logging feature |
| 2026-03-25 | Rewrote procedure doc to match Confluence layout, simplified language |
| 2026-03-12 | Fixed junk column leakage with `fields - _*` on all write paths |
| 2026-03-12 | Fixed decision history to show approved/denied as separate rows |
| 2026-03-12 | Fixed "Expiring This Week" KPI counting approved entries |
| 2026-03-12 | Added Jira ticket naming convention (ISO 8601) |
| 2026-03-12 | Confirmed dashboard does not write to KV store |
