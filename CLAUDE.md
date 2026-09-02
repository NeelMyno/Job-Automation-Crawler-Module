# Job Crawler: repo operating manual

This repo does one thing: find fresh, real job postings from public ATS boards and Hacker News,
filtered to your own target companies and role. It does not draft résumés, cover letters, or
outreach, and it does not track applications. That's intentionally out of scope here.

## What to do

- **Run the crawler:** `python3 pipeline/job-crawler/crawl.py [--write] [--hours N] [--company X]`.
  See `pipeline/job-crawler/README.md` for the full flag list.
- **Point it at a new search:** edit `pipeline/job-crawler/boards.yaml` (companies) and
  `pipeline/job-crawler/filters.yaml` (role titles, keywords, location, and any hard/soft rules;
  read that file's own comments before touching `exclude_patterns`/`flag_patterns`, since what
  belongs in each depends entirely on your own situation).
- **Re-check a lead before applying:** `python3 pipeline/job-crawler/liveness.py --url <URL>`. A
  meaningful fraction of postings are ghosts or already filled by the time you get to them.

## Ground rules

- **Never invent a posting, a field, or a fact.** The crawler only selects and copies real fields
  from real postings; it never fabricates anything. If you're ever asked to extend this (e.g. add
  a new ATS provider), keep that discipline: select and copy, never generate.
- **Never automate a login-walled platform** (LinkedIn and similar). That risks an account ban.
  Reading the free public ATS JSON APIs is fine; logging into anything is not.
- **This repo has no honesty gates, résumé engine, or outreach machinery**, unlike the fuller
  version of this tool. That's deliberate: you're using this for discovery only and handling
  applications yourself. Don't add that machinery back in piecemeal; if you eventually want it,
  the companion full-pipeline release is the place for it, not a bolt-on here.

## Verify before trusting a change

```
cd pipeline/job-crawler && python3 test_liveness.py     # offline, no network, should print OK
python3 pipeline/job-crawler/crawl.py --hours 720        # should return real live postings as JSON
```
