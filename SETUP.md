# SETUP

## 0. Before you start

You don't need to be a developer. You need three things:

1. **The repo on your computer.** On the GitHub page, click the green **Code** button, then
   **Download ZIP**, and unzip it. Or, if you use git, copy the URL from that same button and
   run `git clone <that URL>`.
2. **A terminal, opened in the repo's folder.**
   - Mac: open **Terminal** (press Cmd+Space, type "Terminal"). Type `cd ` (with a space), drag the
     unzipped folder onto the window, and press Return.
   - Windows: open the unzipped folder in File Explorer, click the address bar, type `cmd`, and
     press Enter.
   Every command below runs from this folder (the one that holds `SETUP.md`).
3. **Python 3.** Check with `python3 --version` (Windows: `py --version`). If it says "not found",
   install it from https://www.python.org/downloads/ (on Windows, tick "Add python.exe to PATH" in
   the installer), then close and reopen the terminal.

On Windows, type `py` wherever this guide says `python3`.

## 1. Install

```
python3 -m pip install -r pipeline/job-crawler/requirements.txt
```
That's the one dependency (PyYAML). No accounts and no login for anything in this repo. The only
optional key is a free USAJOBS key (step 4).

## 2. Set your titles: `pipeline/job-crawler/filters.yaml`

Put your own role titles under `include_titles`, for any role in any industry. This is the field
that targets the whole crawler, and board discovery uses it too. It ships empty, and `crawl.py`
won't run until it has at least one title. Matching is a case-insensitive substring of the job
title, so a short, common phrasing catches more. A title of 3 characters or fewer ("rn", "cna")
must match as a whole word, so "rn" does not match "International". Examples:
```yaml
include_titles:
  - data analyst
  - business analyst
```
```yaml
include_titles:
  - registered nurse
  - nurse practitioner
```
```yaml
include_titles:
  - mechanical engineer
  - project engineer
```

**One thing worth reading closely before you fill in `exclude_patterns`:** the file ships with
commented-out examples showing how one requirement can be a dead end for one person and a plus for
another. Someone without a required license or certification (a state nursing license, a CPA, a
professional engineering license) might exclude postings that demand it. Someone who holds it might
**add** the same terms to `include_keywords` instead, so those postings rank higher. This repo has
no opinion on which is right for you; both lists ship empty so you make this call yourself.

## 3. Run it once: job APIs and Hacker News work right away

```
python3 pipeline/job-crawler/crawl.py --hours 720   # wide window for a first run
```

On a fresh clone `boards.yaml` is empty, so this run searches the free job APIs (Himalayas, Jobicy,
Remotive, Remote OK) and the Hacker News "Who is hiring?" thread, and tells you to run a sweep. The
JSON output's `sources` list shows each source, how many jobs it returned and matched, and why any
was skipped (for example "disabled in sources.yaml", or "last fetched 12 min ago" when a source's
rate limit means its previous results were reused).

## 4. Add company boards (no hand-listing)

```
python3 pipeline/job-crawler/discover.py --sweep
```
This harvests company job-board names from the free Common Crawl index, reads each board's public
API, and adds to `boards.yaml` only the companies with an open posting matching your titles. The
first run harvests the Common Crawl index before checking any board, which can take several minutes
and may hit timeouts (HTTP 504) that it retries. Later runs reuse that harvest and check about 150
boards each, in a few minutes, remembering their progress, so run it again to keep going. If the
index stays overloaded, it says so; try again later.

Company boards are read directly on Greenhouse, Lever, Ashby, SmartRecruiters and Workable (the
sweep and `--company` cover these five), and on Workday when you give a careers URL to `--urls`.
iCIMS, Taleo, Oracle and SuccessFactors boards are not read directly: for an employer on one of
those, find its postings with a web search, and pipe any that sit on a supported ATS into `--urls`.

Other ways in:
```
python3 pipeline/job-crawler/discover.py --company "Acme" "Globex"   # named companies
python3 pipeline/job-crawler/discover.py --queries                   # web-search queries for your titles
python3 pipeline/job-crawler/discover.py --urls - < urls.txt         # any job/careers URLs, e.g. those results
```
Add `--dry-run` to see what would be added first. Rows discovery adds carry `source: discover:...`;
it never edits or removes a row you wrote.

Optional: federal jobs. Request a free key at https://developer.usajobs.gov, then set
`USAJOBS_API_KEY` and `USAJOBS_EMAIL` in your environment (never in a file). Other sources can be
switched on or off in `pipeline/job-crawler/sources.yaml`; read each one's terms summary there first.

## 5. Run it for real

```
python3 pipeline/job-crawler/crawl.py --hours 168 --write
```
`--write` also writes a plain markdown table at `pipeline/job-feed.md`, with the source of every row.
If you get zero matches: widen `--hours`, check your `include_titles` match how employers phrase the
role, and run another `discover.py --sweep`.

A small count often just means your jobs are somewhere this crawler can't read. The free feeds are
mostly remote and tech jobs, and many employers (hospitals, schools, government, retailers) use
hiring systems it can't read. The crawl says so in its `hints` when it finds little. Then:
```
python3 pipeline/job-crawler/discover.py --queries
```
prints ready-made web searches, including plain ones like "registered nurse jobs columbus" built
from your `home_metros`. Run them, open the postings you find, and apply there. Feed any result
URLs to `discover.py --urls -` too: those on a supported ATS become boards the crawler reads next
time.

## 6. Before you apply to a lead, re-check it's still live

```
python3 pipeline/job-crawler/liveness.py --url <the posting URL>
```
Some postings are already closed by the time you get to them. This is a quick, free, conservative
check: it only ever says "dead" on strong evidence.

## Verify everything works

```
cd pipeline/job-crawler
for t in test_*.py; do python3 "$t" || echo "FAILED: $t"; done
python3 discover.py --selftest && python3 sources.py --selftest
python3 titles.py --selftest && python3 locations.py --selftest
```
Each should end with `OK`. (The `for` line is for a Mac or Linux terminal; on Windows, run each
`test_*.py` file with `py`.) They are fully offline, with no network calls and no dependency on your own
config being filled in yet.

## Troubleshooting

- **`ModuleNotFoundError: No module named 'yaml'`** → `python3 -m pip install PyYAML`.
- **`pip: command not found`** → use `python3 -m pip ...` as in step 1 (Windows: `py -m pip ...`).
- **`python3: command not found`** → step 0, point 3.
- **"No job titles set"** → step 2.
- **0 matches** → see step 5.
- **A board's `errors` entry says "HTTP 404"** → that board's slug is wrong or the company moved
  ATS. Remove the row, or re-add it with `discover.py --urls` from its current careers page.
- **A source shows `"status": "cached"`** → it was called recently and its terms limit how often it
  may be called; its last results were reused. This is expected.
