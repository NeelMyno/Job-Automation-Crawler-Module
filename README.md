# Job Crawler

A free, real-time job crawler: it pulls fresh postings straight from your target companies' public
ATS boards (Greenhouse / Lever / Ashby) and the monthly Hacker News "Who is hiring?" thread,
filtered to your own role and location. No résumé/cover-letter generation, no application tracker,
no outreach — **just job discovery.** You fill out and submit applications yourself; this just
finds them for you, fast, before they're buried under a week of new listings.

**$0. No API keys. No login. No scraping of any login-walled platform.** It reads the public ATS
JSON that companies already publish for their own careers pages.

## Quick start

See `SETUP.md` for the full walkthrough. The short version:

```
pip install -r pipeline/job-crawler/requirements.txt
# edit pipeline/job-crawler/boards.yaml   -> your target companies
# edit pipeline/job-crawler/filters.yaml  -> your role titles
python3 pipeline/job-crawler/crawl.py --hours 720
```

That prints fresh, real, live postings as JSON. Add `--write` to also get a readable markdown
table at `pipeline/job-feed.md`.

## What's in here

- `pipeline/job-crawler/crawl.py` — the crawler itself.
- `pipeline/job-crawler/hn.py` — the Hacker News "Who's Hiring" collector it also pulls from.
- `pipeline/job-crawler/liveness.py` + `test_liveness.py` — an optional ghost-job re-checker: before
  you spend time on a lead, re-verify the posting is actually still live.
- `pipeline/job-crawler/boards.yaml` — your target companies. **Edit this.**
- `pipeline/job-crawler/filters.yaml` — your role titles, keywords, and location rules. **Edit
  this — `include_titles` is the field that actually targets the crawler at your search.**
- `pipeline/job-crawler/README.md` — the crawler's own detailed docs (setup, options, the
  sponsorship-flag mechanism, how to add a company).

## This is a generic tool, not tuned to any one field

Nothing here assumes a design/product role, or any particular visa/work-authorization situation.
`filters.yaml`'s comments walk through exactly what to edit for your own search — including a note
specifically relevant if you're targeting cleared/defense-adjacent roles (see SETUP.md).
