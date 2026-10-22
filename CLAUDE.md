# Job Crawler: repo operating manual

This repo does one thing: find fresh, real job postings for the user's own titles, in any role and
any industry, from company job boards, free job APIs, and Hacker News. It does not draft résumés,
cover letters, or outreach, it does not fill forms, and it does not track applications. That's
intentionally out of scope here.

## How discovery works (read before describing this repo to the user)

The crawler is NOT tied to a fixed set of companies or job boards. Three kinds of source:

1. **Company boards** in `pipeline/job-crawler/boards.yaml`. The sweep and `--company` cover
   Greenhouse, Lever, Ashby, SmartRecruiters and Workable; Workday boards come in only through
   `--urls`. iCIMS, Taleo, Oracle and SuccessFactors are not read directly: for an employer on one
   of those, find its postings with your web search and pipe any URL on a supported ATS into
   `--urls`. Say this plainly when the user asks what is covered. The file starts empty; you (the
   agent) fill it with `discover.py`, never by guessing slugs:
   - `discover.py --sweep`: harvests boards from the Common Crawl index and adds those with postings
     matching the user's titles. The default way to build the list. The FIRST run harvests the
     index before checking any board, which can take several minutes and may hit timeouts it
     retries; tell the user before you start it. Later runs check about 150 boards in a few minutes
     and remember where they stopped; run it again to continue. The
     index is a free public service and sometimes times out: when it does, research with your web
     search which companies hire for the user's titles, pass them to `--company`, and retry the
     sweep later. Never leave the user with an empty list because the index was down.
   - `discover.py --company "Name" ...`: when the user names companies.
   - `discover.py --queries`, then run those searches with your web search tool and pipe every result
     URL into `discover.py --urls -`. If your search tool ignores `site:`, use the plain
     `"<title> jobs <company> careers"` queries it also prints.
2. **Free job APIs** in `pipeline/job-crawler/sources.yaml` (Himalayas, Jobicy, Remotive, Remote OK
   on by default; USAJOBS when its key is in the environment; The Muse and We Work Remotely off until
   the user turns them on).
3. **Hacker News "Who is hiring?"**.

When you report what was searched, read the `sources` list in `crawl.py`'s JSON output: it names
every source, whether it ran, how many jobs it returned, and why any was skipped. Never say the repo
"only fetches from some job boards".

## What to do

- **Set titles first:** `pipeline/job-crawler/filters.yaml` `include_titles` (ask the user for their
  role titles if it is empty; `crawl.py` refuses to run without them).
- **Run the crawler:** `python3 pipeline/job-crawler/crawl.py [--write] [--hours N] [--company X]`.
  See `pipeline/job-crawler/README.md` for every flag.
- **Grow the board list:** `python3 pipeline/job-crawler/discover.py --sweep` (or `--company`, `--urls`).
- **Re-check a lead before the user applies:** `python3 pipeline/job-crawler/liveness.py --url <URL>`.

## Ground rules

- **Never invent a posting, a field, or a fact.** The crawler and discovery only select and copy
  real fields from real postings; a board is added only after its own API answered. If you extend
  this (a new ATS or job API), keep that discipline: select and copy, never generate.
- **Respect each source's terms.** Limits are enforced in `sources.py`; never lower a floor, never
  remove the source name from feed rows, and never publish the feed (several terms forbid
  re-posting the jobs). Leave a new source `enabled: false` until its terms are read.
- **Never automate a login-walled platform or a logged-in account** (LinkedIn, Indeed, Glassdoor,
  email, social accounts). That risks a ban. Reading free public JSON APIs is fine; logging into
  anything is not. Keys live in environment variables, never in a file.
- **This repo has no honesty gates, résumé engine, form filling, or outreach machinery**, unlike the
  fuller version of this tool. That's deliberate. Don't add that machinery back in piecemeal; the
  companion full-pipeline release is the place for it.

## Verify before trusting a change

```
cd pipeline/job-crawler
for t in test_*.py; do python3 "$t" || echo "FAILED: $t"; done                   # offline, each ends OK
python3 discover.py --selftest && python3 sources.py --selftest                     # offline
python3 titles.py --selftest && python3 locations.py --selftest                     # offline
python3 crawl.py --hours 720        # real live postings as JSON, with the `sources` report
```
