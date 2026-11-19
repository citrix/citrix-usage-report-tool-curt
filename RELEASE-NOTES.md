# Release notes — Citrix Usage Report Tool (CURT)

## 1.16.0

**Faster on large sites. No figure changes.**

- **Export this site for combining is about five times faster**, and needs
  far less memory. `merge-export.json` is now written a session at a time.
  Before, the whole file was built in memory first, which took about
  25 minutes for 1.5 million sessions and came close to the largest text
  Windows PowerShell can hold. The file's contents are the same, field for
  field.
- The export is written under a temporary name and renamed when complete.
  A run that is stopped or fails part-way leaves the previous export, if
  there was one, untouched, and never leaves half a file.
- The anonymized keys for each user are worked out faster.
- The application breakdown uses less memory on sites with millions of
  sessions.

## 1.15.2

**Accuracy**

- **Application launches are counted in the period they happened in.** A
  launch was counted whenever its session was open in the period, so a
  desktop left running for two months put every launch of those two months
  into the 7-day figure. The script now reads each launch's start time and
  counts it only inside the period. "Unique users" for an application is
  unchanged: people whose session ran it and was open in the period.
- A connection or an application launch that Citrix returns twice, because
  the site changed while it was being read, is counted once. Sessions
  already were.

**Large sites**

- **About half the memory while reading.** Each page of results is converted
  to what the report keeps as it arrives. Before, the script held every raw
  record and every converted one together: about 4.4 GB for each million
  sessions.
- Two session fields that nothing used are no longer requested, so every
  page of sessions is smaller.
- Connections and application launches from before the period (they belong
  to sessions still open in it) are read a day at a time, like the rest.
  Before, they were one long query, which was the slowest single read of a
  short report on a site with long history.

## 1.15.1

**Accuracy**

- **Fixed: a combined report counted sessions that an export saw still open
  as running until the day the exports were combined.** Exports made on
  1 October and combined in mid-November had every such session running for
  six more weeks, which raised the peak and the percentiles and put those
  users on every day of the trend. A combined report is now measured back
  from the end of its newest export, and says so ("Data runs up to"). A
  session open at an older export is counted up to that export's end.
  **Combined reports made with an earlier version should be made again**; the
  exports themselves are fine and do not need re-running.
- **Fixed: when one session appeared in two exports of the same site, the
  copy from whichever file was read first was kept.** If that was the older
  export, the session stayed open even though the newer export recorded its
  end. The copy that records the end is now kept.
- A combined report now warns, naming the site, when a site's newest export
  stops more than a day before the period ends. Its last days are missing, so
  the total is a lower bound.
- A combined report now warns when a site has accounts with no directory
  security identifier. Those accounts cannot be matched across sites: their
  sessions are in every concurrency figure, but they are not in the
  unique-user total, and the report says by how many it may be low.
- **Fixed: a dropped connection or a timeout was not retried.** Only HTTP
  error codes were. One network blip in a long read could end the run. It is
  now retried like any other passing fault, once the server has answered at
  least once; a wrong host name is still reported at once.
- **Fixed: a response that was not Monitor data** (a proxy or gateway
  answering with a web page) was read as the last page of the query, and the
  rest of that day's sessions were dropped with no warning. It is now asked
  for again, and the run fails with a clear message if it keeps happening.
- **Fixed: on a machine set to the Thai or Arabic (Saudi) calendar, the
  dates in the query sent to Citrix carried that calendar's year**, so the
  wrong period was asked for.
- The business-hours peak ("Highest at any single moment") is now exact. It
  was the highest of the 15-minute samples, so it could miss a peak between
  two samples and show business hours below the overall peak.
- Percentiles in the report are rounded **up** to a whole session. The report
  says concurrency was "at or below" the figure, so 6288.1 must read 6289;
  before, it was rounded to the nearest, and a half went to the even number.
- `summary.csv` writes decimals with a full stop on every machine. A German
  or French machine wrote `12,5`. It also gains `PartialCoverage`,
  `WindowStartUtc` and `WindowEndUtc` columns, at the end.
- `data.json` gains `DataAsOfUtc` and, for each window of a combined report,
  `PartialCoverage`.

**Security and privacy**

- The link to the next page of results is no longer followed to a different
  server. It carries the run's sign-in, and over plain HTTP a tampered
  response could have sent that sign-in elsewhere. Only the path is taken
  from the link; the server is always the one the run was pointed at. This
  also helps sites behind a load balancer whose links name an internal host.
- Over plain HTTP with a supplied account, the script no longer answers a
  Basic sign-in request, which would send the password as readable text.
- **Fixed: `-Anonymize` on a report combined from exports that were not
  anonymized still printed real user principal names** in the list of name
  matches and in the run log. Both now show pseudonyms. The
  `consolidated-identity-map.csv` from such a run holds real security
  identifiers, as it must to be useful; the log now says to keep it and not
  send it.
- Values written to the CSV files that begin with `=`, `+`, `-` or `@` get a
  leading apostrophe, so a spreadsheet shows them as text and does not run
  them as a formula. Find a user reads them back unchanged.
- The key file (`anonymization-salt.txt`) and `identity-map.csv` are now
  readable only by the person who created them, the system and
  administrators, wherever the output folder is.
- One damaged file no longer stops a whole combine: a merge export with an
  unreadable version or window is rejected by name and the rest carry on.

## 1.15.0

**Faster on large sites**

- **Sessions are read a day at a time, four requests at once.** Before, one
  long query read every session in order, and on-premises sites slow down
  the further into a query they get. On a test of 50,000 sessions shaped
  like a large site, the read took about 12 seconds instead of 145. The new
  `-FetchConcurrency` option (1 to 8, default 4) sets how many requests go
  at once; use 1 if the Delivery Controller is under strain. A request that
  fails is retried as before, and its day carries on from where it stopped.
- Connections and application instances are read the same way, a day at a
  time.
- **The analysis is about eight times faster.** For each period, the script
  now works out each session's place in the period once, and every figure
  reuses it. On one site's 489,268 sessions, a 30-day analysis took 85
  seconds instead of nearly 12 minutes. Every figure is the same as before.
- `sessions.csv` is written directly, much faster on sites with millions of
  sessions. Its content is unchanged.

## 1.14.4

**Large sites**

- **Fixed:** on large sites the peak, the percentiles, the unique users and
  the total sessions could be far below the real figures, and nearly the same
  for 30, 60 and 90 days. The script stopped reading after 5000 pages, and
  some sites send only 100 records a page, so it kept an arbitrary 500,000
  sessions. One site reported a peak of about 2,500 against a true 6,925. The
  report carried a "not fetched completely" banner, but the figures looked
  complete. **Reports from large sites made with an earlier version should be
  run again**, as should their exports for combining.
- The same limit cut short the application breakdown on large sites. It now
  reads up to 25 million records per query, whatever the page size.
- If session data is ever incomplete, every headline figure is marked
  **Lower bound**, `summary.csv` has a new `SessionsIncomplete` column, and
  the console says so at the end of the run.
- A combined report now warns, naming the site, when one site's export did
  not read every session. Before, it carried no warning at all.
- **Find a user** says "cannot confirm" rather than "not counted" for a run
  that did not read every session, including reports made with earlier
  versions.
- A brief network or server outage no longer ends a long run: each page is
  tried up to 8 times, over about two minutes, rather than 5.
- A long Citrix Cloud run can renew its sign-in as often as it needs to.
- Exports for combining are written compactly: about half the size, and
  quicker to write and read, which matters on sites with millions of
  sessions.
- `daily-trend.csv` writes dates as `2026-10-04` rather than in the
  machine's own format, which a reader elsewhere could misread.

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
