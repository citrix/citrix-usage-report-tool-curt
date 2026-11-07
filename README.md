# Citrix Usage Report Tool (CURT)

One PowerShell script that reads the Citrix Monitor Service and produces a
self-contained HTML report of **unique users** and **concurrent sessions**
over the periods you choose.

- **Read-only.** It reads Citrix Monitor data and changes nothing. The only
  call that is not a read is the Citrix Cloud sign-in, which exchanges your
  API client's ID and secret for an access token.
- **Installs nothing.** One `.ps1` file, no modules, no agents. Windows
  PowerShell 5.1, which is built into Windows 10, Windows 11 and Windows
  Server.
- **Report in English** on any Windows language.

## Before you start

You need one of:

- **On-premises:** a **Delivery Controller** hostname (not a standalone
  Director server), and an account with read access to Monitor data. The
  built-in **Read Only Administrator** role is enough.
- **Citrix Cloud:** your **customer ID**, plus an **API client ID and secret**
  with read access, from Identity and Access Management → API Access.

The machine you run it on needs network access to the Delivery Controller, or
to `api.cloud.com` (Commercial), `api.citrixcloud.jp` (Japan) or `api.cloud.us`
(Government).

## Download and run

1. Save `CitrixUsageReport.ps1` as a file somewhere on the machine. Run it
   from that saved file, not by pasting its contents into a PowerShell window.
2. Unblock it: right-click → **Properties** → tick **Unblock**, or run
   `Unblock-File .\CitrixUsageReport.ps1`.
3. Open PowerShell in that folder and run:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\CitrixUsageReport.ps1
```

`-ExecutionPolicy Bypass` applies only to that one PowerShell process and
changes no setting on the machine. Double-clicking the file does not run it;
Windows opens `.ps1` files in an editor.

The dialog opens. Choose your environment, fill in the fields it shows, and
click **Run Report**. The window stays open and shows progress, a log and a
**Cancel** button. When the run finishes, click **Open report**.

**Reporting windows** defaults to `30` days, the quickest run. Enter
`30,60,90` to compare several periods. Each extra window adds to the run time.

### Try it without Citrix first

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\CitrixUsageReport.ps1 -DemoData -NoGui -OutputPath .\Demo
```

This uses synthetic data, contacts nothing, and watermarks the report as demo
data.

### From the command line

On-premises, with a different account:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -Command ".\CitrixUsageReport.ps1 -DeliveryController ddc01.example.com -Credential (Get-Credential) -NoGui -Days 30,60,90 -OutputPath C:\Reports -IncludeDeliveryGroups -IncludeTrend"
```

Citrix Cloud:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -Command ".\CitrixUsageReport.ps1 -Environment CloudCommercial -CustomerId <id> -ClientId <client-id> -ClientSecret (Read-Host 'Secret' -AsSecureString) -NoGui -Days 30,60,90 -OutputPath C:\Reports"
```

Use `-Command`, not `-File`, when passing a credential or secret. Write
`-Days 30,60,90` with no spaces. On the command line, `-Days` defaults to
`30,60,90`.

## Options

| Option | Effect |
|---|---|
| `-Environment` | `OnPremises` (default), `CloudCommercial`, `CloudJapan` or `CloudGovernment`. |
| `-DeliveryController <host>` | On-premises: the controller to read from. |
| `-Protocol Https\|Http` | How Monitor is published on the controller. Defaults to `Https`. |
| `-Credential` | On-premises account. Defaults to the signed-in user. |
| `-CustomerId`, `-ClientId`, `-ClientSecret` | Citrix Cloud credentials. |
| `-Days 30,60,90` | Reporting windows, in days. |
| `-OutputPath <folder>` | Where reports go. Defaults to `Documents\CitrixUsageReport`. |
| `-IncludeDeliveryGroups` | Unique users and peak concurrency per delivery group. |
| `-IncludeApplications` | Desktop and application split, per-application table. Slower. |
| `-IncludeClientDevices` | Client devices and Workspace app versions. Slower. |
| `-IncludeTrend` | Day-by-day unique users and peak concurrency. |
| `-Anonymize` | Replace names with `User-0001` style pseudonyms. Counts are unchanged. |
| `-ExportRawData` | Also write `data.json`, `summary.csv`, `sessions.csv` and `daily-trend.csv`. |
| `-FetchConcurrency 1-8` | Requests to the Monitor Service at once. Default 4. Use 1 to send one at a time. |
| `-NoGui` | Run without the dialog. |
| `-DemoData` | Synthetic data. Contacts nothing. |

Run `Get-Help .\CitrixUsageReport.ps1 -Full` for every parameter.

## What you get

Each run creates a folder such as `CitrixUsageReport-20261106-091500` under the
output folder:

| File | When | Send it back? |
|---|---|---|
| `CitrixUsageReport.html` | Always. Opens offline; **Save as PDF** is built in. | Yes |
| `usage-report.log` | Always. Credentials and tokens are removed. | Yes |
| `data.json`, `summary.csv`, `sessions.csv`, `daily-trend.csv` | `-ExportRawData` | Yes |
| `merge-export.json` | Export for combining (see below) | Yes |
| `identity-map.csv` | `-Anonymize`. Maps pseudonyms back to real names. | **No, keep it** |
| `anonymization-salt.txt` | Export for combining. The shared key file. | **No, keep it** |

Also keep, and never send: your identity mapping file, and any page saved by
the find-a-user helper.

### Reading the report

- **Concurrency counts sessions, not people.** One person with a desktop and
  two published applications open counts as three sessions.
- **The distribution matters more than the peak.** "Highest at any single
  moment" is one instant. "19 times out of 20, at or below" is what the site
  ran at nearly all the time.
- **A truncation banner means a lower bound.** Citrix keeps session history
  for 90 days on Premium, 31 on Advanced and 7 on other editions. A window
  longer than that is marked, and the true figure is higher than shown.
- **"Lower bound: session data incomplete"** under the figures means the
  script could not read every session. It is very unlikely from 1.14.4 on.
  If you see it, send `usage-report.log` with the report.

## Combining several sites

Adding up separate site reports **counts anyone who uses two sites twice**.
To get one correct total:

1. **At each site**, tick **Export this site for combining** (`-ExportForMerge`).
   At the first site click **Create new** to make the shared key file; at
   every other site click **Use existing** and choose a copy of that same
   file. The key fingerprints must match.
2. **On one machine**, choose **Combine saved exports from several sites (no
   connection)**, add the `merge-export.json` files, and click **Run Report**:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\CitrixUsageReport.ps1 -Merge C:\exports -Days 30,90,180 -OutputPath C:\Reports
```

Keep the key file safely. Exports made with different key files cannot be
combined. If retention is short, export monthly and combine the exports later
to cover a longer period.

People in different Active Directory forests are matched by user principal
name only if you turn on **Also match people across directories by user
principal name** (`-CrossForestKey Upn`). It is off by default.

## Identity mapping: one person, different accounts

If one person has differently named accounts in each site, such as
`AA\AAJaneDo` and `AB\ABJaneDo`, nothing in the data links them. An **identity
mapping file** tells the tool which accounts belong to one person.

Click **New template** beside the identity mapping field to save a starting
CSV. It explains each column in `#` comment lines and contains commented-out
examples. The format is:

```csv
Person,Account,Note
,AA\AA*,site AA naming convention
,AB\AB*,site AB naming convention
JaneDo,AB\ABJDoe,exception to the convention
```

- `Account` can be `DOMAIN\name`, a bare name, a principal name or a SID.
- A row with one `*` and a blank `Person` is a pattern. Whatever the `*`
  matches identifies the person, so `AA\AAJaneDo` and `AB\ABJaneDo` both become
  `JaneDo`.
- A row naming one account wins over a pattern. Its `Person` must exactly
  match what the pattern produces (`JaneDo`, not `Jane Do`).
- Comma or semicolon separators, and Excel's CSV saves in any language, all
  work.

Use it **at each site when exporting**, where every row type works, or **when
combining** existing exports, where only principal-name rows apply. Use each
row at one stage, not both. Command line: `-IdentityMapPath <file>`.

The file pairs real people with their accounts. Never send it back with a
report.

## Finding a user in saved reports

To confirm whether a person was counted, choose **Find a user in saved reports
(no connection)**, enter part of a name, username or principal name, and click
**Search**. Each matching account shows one of:

- **Found: N session(s), counted in the report.**
- **Found, not counted:** no session inside the report's windows.
- **Found: cannot confirm**, with the reason. Usually the run did not save
  its sessions; run again with **Export raw data**.

Click **Save page for leadership...** to save a one-page summary you can
print to PDF. It contains real names and is for internal use only.

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\CitrixUsageReport.ps1 -FindUser 'jane.doe@example.com' -FindUserPage C:\Temp\user-check.html
```

This reads saved files only and contacts nothing. Only runs made with
**Anonymize** can be searched, and **Export raw data** is needed to confirm
sessions. By default only the latest run for each site is searched. Tick
**Show older runs too**, or add `-AllRuns`, to include earlier runs.

## Troubleshooting

| Message | What to do |
|---|---|
| Script will not start, mentions `about_Execution_Policies` | Run it with `-ExecutionPolicy Bypass` exactly as shown above. |
| **401** | The account was rejected. Check it has Monitor read access, or supply a different account. |
| **403** | The account signed in but has no Monitor read access. |
| **404 on the Monitor OData endpoint** | The host is not a Delivery Controller, or Monitor is not published over HTTPS. Point at a controller, or try `-Protocol Http`. |
| **"Citrix Cloud rejected the credentials"** | Check the customer ID, client ID and secret. Secrets are shown once; create a new one if it has been lost. |
| **TLS certificate trust failure** | This machine does not trust the controller's certificate. Run from a domain-joined machine, or install the issuing root certificate. |
| **"Set-ReportCancellation is not recognized"** | Fixed in 1.14.3. On older versions, save the script as a file and run that file. |
| **Peak far lower than Director shows, and the same for 30, 60 and 90 days** | Fixed in 1.14.4. Earlier versions stopped reading after 500,000 records on some large sites. Run the report again with 1.14.4. |
| **"No sessions were returned"** | The run worked but found no sessions. Check the account's access scope and compare with Director for the same period. |

Every error also appears in `usage-report.log` in the run folder.

### Exit codes

`0` the report was written · `1` the run failed (see `usage-report.log`) ·
`2` canceled, or the dialog was closed before a run finished.

## Notes

Funnel Display and Public Sans are embedded in the report under the SIL Open
Font License. Citrix and Citrix Virtual Apps and Desktops are trademarks of
Cloud Software Group, Inc.
