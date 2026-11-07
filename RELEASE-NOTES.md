# Release notes — Citrix Usage Report Tool (CURT)

## 1.14.3 — first public release

This release is the first published at
<https://github.com/citrix/citrix-usage-report-tool-curt>. It includes
everything below since 1.10.0.

**Fixes**

- Clicking **Run Report** no longer fails with "The term
  'Set-ReportCancellation' is not recognized" when the script was started
  from its text rather than a saved file, for example pasted into an untitled
  PowerShell ISE tab or run by a deployment tool. This affected 1.11.0 to
  1.14.2.
- `Get-Help .\CitrixUsageReport.ps1 -Full` now shows the script's full help,
  with a description of every parameter and the examples. Earlier versions
  showed only the parameter list.

## 1.14.2

**Reports read the same in any Windows language**

- The report and its files are always in English, with English dates,
  English time zone names and a decimal point in CSV numbers.
- **Fixed:** on Windows set to a language that uses a decimal comma (French,
  German, Dutch and most of Europe), the line charts drew wrongly: the line
  stuck to the top edge and the shaded area became a wedge. The figures and
  bar charts were correct. **Reports made on those machines with an earlier
  version should be run again.**
- Merge exports and identity mapping files are read correctly whatever
  language the machine uses, including Excel CSV files saved with semicolons.

## 1.14.1

- The dialog and the console prompt now default to a single **30-day**
  window, the quickest run. Enter `30,60,90` to compare several periods. On
  the command line, `-Days` still defaults to `30,60,90`.

## 1.14.0

**Find a user in saved reports**

A new **Find a user in saved reports (no connection)** option in the dialog,
and `-FindUser` on the command line, answer "was this person counted?"
from reports already on your machine. It contacts nothing and writes no log.

- Search by part of a name, username or principal name.
- Each account found is **counted** (with its number of sessions), **not
  counted** (no session in the report's windows) or **cannot confirm**
  (with the reason).
- **Save page for leadership...** (`-FindUserPage`) saves a one-page summary,
  ready to print to PDF. It contains real names and is for internal use only.
- Searches the latest run for each site; **Show older runs too** (`-AllRuns`)
  includes earlier runs.

## 1.13.0

- A **New template** button beside each identity mapping field saves a
  starting CSV. It explains each column and includes commented-out examples
  of every kind of row.

## 1.12.0

**Identity mapping across sites**

When one person has differently named accounts in different sites, such as
`AA\AAJaneDo` and `AB\ABJaneDo`, a combined report used to count them twice.
An identity mapping file (`-IdentityMapPath`, or the field in the dialog)
lists which accounts belong to one person.

- Accounts can be given as `DOMAIN\name`, a bare name, a principal name or a
  SID.
- Patterns such as `AA\AA*` cover a whole naming convention in one row, and
  rows naming a single account handle the exceptions.
- It can be applied at each site when exporting, or when combining exports
  you already have.
- The run log refers to rows by number and never names an account.
- `consolidated-identity-map.csv` shows which accounts were joined by the
  mapping.

## 1.11.2

- **Fixed:** "AuthorizationManager check failed" as soon as **Run Report**
  was clicked, on machines where Group Policy sets the execution policy to
  `Unrestricted` and the file was not unblocked.
- Errors shown in the run page's activity log have credentials and tokens
  removed, as the log file already did.

## 1.11.0

**The dialog stays open through the run**

- Clicking **Run Report** switches the same window to a run page showing each
  phase, a progress bar, the elapsed time and a live activity log.
- **Cancel** works throughout and takes effect within about a second. Files
  the run had not finished writing are deleted; the run folder and its log
  are kept.
- Closing the window during a run asks first.
- When the run finishes: **Open report**, **Open folder** or **Run another**.

## 1.10.0

- Analysis is about 2.8 times faster on large sites, with identical figures.
