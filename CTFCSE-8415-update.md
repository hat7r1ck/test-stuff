# CTFCSE-8415 - Automate Daily Standup Report Creation
## Research & Testing Update - 3/26/2026
### Performed by: Jeff Morris

---

## Objective

Automate the CTFC Site Report currently sent 3x daily (end of Bangalore, Charlotte, and Chandler shifts) via SecurityMonitoring@wellsfargo.com to the G=CTFC_Site_Report distribution list. Report requires data from XSOAR (case management metrics), Jira (S&E Tuning and Break/Fix tickets from CTFC-SE dashboard), and Confluence (on-call schedule/contact sheet links).

---

## Approaches Tested

### 1. Power Automate - Jira Connector
**Result: BLOCKED by DLP**
- The Jira connector is classified as blocked in our tenant's DLP policy.
- Flow is immediately suspended upon saving with Jira connector actions.

### 2. Power Automate - HTTP Connector (Direct Jira REST API Call)
**Result: BLOCKED by DLP**
- Attempted to use the HTTP action to make raw REST API calls to securityjira.wellsfargo.net with a PAT.
- The HTTP connector is also blocked by DLP policy in our environment.

### 3. Power Automate - SharePoint Connector
**Result: BLOCKED by DLP**
- SharePoint connector is blocked in our Power Automate environment, eliminating the intermediary file approach (script writes to SharePoint, Power Automate reads and emails).

### 4. Power BI Service - Web.Contents() Direct API Call (No Gateway)
**Result: FAILED - Network Unreachable**
- Created a Dataflow Gen1 in the Cyber Security Operations - PROD workspace.
- Used Web.Contents() in Power Query to call https://securityjira.wellsfargo.net/rest/api/2/search with Bearer token authentication.
- Connection failed: "The connection could not be created. This may mean it is not accessible from this network or at this URI or that a gateway is required to access this data source."
- Root cause: Power BI cloud service cannot reach on-premises/internal resources without a data gateway.

### 5. Power BI Service - Web.Contents() via On-Premises Gateway (BICC_GTWY_SILAS1)
**Result: FAILED - No Gateway Permissions**
- Attempted to route the same Web.Contents() call through the available on-premises gateway (BICC_GTWY_SILAS1).
- Error: "The connection you are trying to use has not been shared with you on the data gateway."
- The gateway exists and is functional, but CTFC team members do not have permission to create new data source connections on it.

### 6. Power BI Desktop Application
**Result: NOT AVAILABLE**
- Power BI Desktop is not installed and is not available for installation on CTFC workstations.
- Desktop would allow direct API calls from the local network without a gateway, then publish to the service. Not an option without the application.

### 7. Microsoft Fabric Notebooks
**Result: NOT AVAILABLE**
- Investigated how the CTO Platform Support team gets Jira data into Power BI. They use a Fabric Pipeline with Notebooks (Python) that call the Jira REST API directly (df_Jira_Kanban_full_refresh pipeline).
- This approach works and is actively refreshing Jira data daily in the CTO Platform Support PROD workspace.
- However, the Notebook item type is not enabled/available in the Cyber Security Operations - PROD workspace. Cannot replicate this approach without Fabric Notebook capability being enabled.

### 8. XSOAR Automation (Playbooks/Scripts)
**Result: NOT PERMITTED**
- XSOAR has native capabilities for scheduled jobs, API calls (including Jira integration), HTML email formatting, and email delivery.
- However, CTFC engineers (including lead/T2 level) do not have permissions to create or run automation scripts, playbooks, or access the XSOAR API.

### 9. Local Scheduled Scripts (Python/PowerShell via Windows Task Scheduler)
**Result: NOT VIABLE**
- Could technically run a local script to pull from Jira/XSOAR APIs and push data to OneDrive/SharePoint for Power BI consumption.
- Not viable due to hybrid work schedule (4x10 shifts) and 3 days per week with no one available to ensure the machine is running. Report must send 3x daily, 7 days a week, including days when the automating engineer is not on site.

---

## What IS Working / Available

| Capability | Status |
|---|---|
| Power Automate - Outlook/Email connector | WORKING - can send scheduled/recurring emails |
| Power BI - XSOAR data (existing) | WORKING - xsoar_data exists in Cyber Security Operations workspace with ~60 day lookback |
| Power BI - Jira data (CTO Platform team) | EXISTS - Jira data flows into CTO Platform Support PROD workspace via Fabric Notebooks/Pipelines, actively refreshing |
| Power BI - Pro License | CONFIRMED |
| Jira PAT | AVAILABLE |

---

## Remaining Options to Test

### Option A: Power Automate Desktop (RPA)
Power Automate Desktop flows run locally on a machine and can execute HTTP requests, run PowerShell/Python scripts, and interact with web applications. Desktop flows operate outside of cloud DLP policies for certain actions. If Power Automate Desktop is available on CTFC workstations:
- A desktop flow could call the Jira REST API locally, format the data, and pass it back to a cloud flow for email delivery.
- **Blocker**: Requires the Power Automate Process license for unattended runs (runs without a user logged in). Also requires a machine that stays on 24/7, same issue as Option 9 above.
- **Worth testing**: Whether Power Automate Desktop is even installed/available on workstations.

### Option B: Power BI - Cross-Workspace Semantic Model Reference
The CTO Platform Support team already has Jira data in Power BI. If their semantic model is accessible:
- A report in the Cyber Security Operations workspace could connect to the CTO Platform's Jira semantic model as a live connection.
- No new data pipeline needed. Just reference their existing Jira data.
- **How to test**: In Cyber Security Operations workspace, click + New Item > Report > Pick a published semantic model, then search for the Jira datasets from CTO Platform Support workspace.
- **Potential blocker**: The CTO team's semantic model may not have Build permission granted to CTFC users. May need them to share access.

### Option C: Request Gateway Access (BICC_GTWY_SILAS1)
- Request that the BICC gateway administrator add CTFC users (or the SecurityMonitoring service account) as authorized users on the on-premises data gateway.
- This would unblock Power BI Dataflow Gen1 from reaching securityjira.wellsfargo.net.
- Unknown turnaround time and approval path.

### Option D: Request Fabric Notebook Capability
- Request that the workspace administrator enable Fabric Notebook items in the Cyber Security Operations - PROD workspace.
- This would allow replication of the proven CTO Platform approach (Python notebook calling Jira API on schedule).
- This is the most scalable and maintainable long-term solution.

---

## Recommendation

The CTFC Site Report automation is blocked by permissions and access restrictions, not by technical limitations. The technology and architecture to fully automate this report already exists and is proven within the organization (CTO Platform Support team's Jira pipeline). The CTFC team is unable to replicate this approach due to:

1. DLP policies blocking all useful Power Automate connectors (Jira, HTTP, SharePoint)
2. No gateway permissions for Power BI to reach internal APIs
3. No Fabric Notebook capability in the CTFC workspace
4. No XSOAR automation permissions despite being lead-level engineers
5. No Power BI Desktop availability

**To unblock this work, at least one of the following access grants is needed:**

| Access Request | Unblocks | Effort to Implement |
|---|---|---|
| Fabric Notebook in Cyber Security Ops workspace | Full automation via Python + Pipeline (proven pattern) | Workspace admin config change |
| Gateway permissions (BICC_GTWY_SILAS1) | Power BI Dataflow Gen1 pulling from Jira/XSOAR APIs | Gateway admin adds data source |
| Power BI Desktop installation | Local development, publish to service with gateway refresh | Software request |
| DLP exception for HTTP connector in CTFC environment | Power Automate direct API calls to Jira | DLP policy change (unlikely) |
| XSOAR automation/API permissions for CTFC leads | Native XSOAR scheduled job handling everything | XSOAR admin role change |

Without at least one of these, full automation of the CTFC Site Report is not achievable with the tools currently available to the CTFC team.
