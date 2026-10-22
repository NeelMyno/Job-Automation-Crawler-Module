# Job Crawler

A free job crawler for **any role, in any industry, at any company**. It finds fresh, real postings
the day they go up, from three kinds of source, all filtered by your own titles and location:

- **Company job boards** on Greenhouse, Lever, Ashby, SmartRecruiters and Workable (found by a
  sweep of the free Common Crawl index or by company name), plus Workday boards you point it at
  with a careers URL. You don't hand-list companies: `discover.py` finds the boards hiring for your
  titles and checks each one live first. iCIMS, Taleo, Oracle and SuccessFactors are not read
  directly; for employers on those, find postings with a web search (see
  `pipeline/job-crawler/README.md`).
- **Free job APIs**: Himalayas, Jobicy, Remotive and Remote OK out of the box, USAJOBS with a free
  key, and more you can switch on in `sources.yaml`.
- **Hacker News "Who is hiring?"**, the monthly thread.

No résumé/cover-letter generation, no application tracker, no outreach. **Just job discovery.** You
fill out and submit applications yourself; this finds them for you, fast, before they're buried.

**$0. No login. No scraping of any login-walled platform.** It reads public JSON that companies and
job sites publish. Keys are optional and free (USAJOBS for federal jobs, The Muse if you switch it
on), read from your environment, never from a file.

**Where you can work.** `filters.yaml` also holds your location rules: `require_us` (drop postings
outside the US), `remote_ok`, `onsite_ok` and `home_metros` (for example remote, or on site only
near Denver). The crawl report says how many postings each rule removed.

**When it finds little.** The free job APIs lean remote and tech, and many employers (hospitals,
schools, government, retailers) use hiring systems this crawler can't read. A crawl with few matches
prints a plain hint, and `discover.py --queries` prints ready-made web searches for your titles and
places, so you can find those postings and apply on the employer's own site.

## Quick start

See `SETUP.md` for the full walkthrough, starting with "Before you start" (getting the repo,
opening a terminal in its folder, installing Python 3). The short version, run from the repo's
folder (on Windows, type `py` for `python3`):

```
python3 -m pip install -r pipeline/job-crawler/requirements.txt
# edit pipeline/job-crawler/filters.yaml  -> include_titles: your role titles (any field)
python3 pipeline/job-crawler/crawl.py --hours 720         # job APIs + HN work right away
python3 pipeline/job-crawler/discover.py --sweep          # then add company boards hiring for your titles
#   the first sweep harvests the Common Crawl index first: several minutes, with retried timeouts;
#   later sweeps check about 150 boards each in a few minutes
python3 pipeline/job-crawler/crawl.py --hours 720 --write # now with those boards; writes pipeline/job-feed.md
```

The JSON output lists every source it searched, how many jobs each returned, and which were skipped
and why, so you can always see exactly what was covered.

## What's in here

- `pipeline/job-crawler/crawl.py`: the crawler itself.
- `pipeline/job-crawler/discover.py`: finds company boards and adds the checked ones to `boards.yaml`.
- `pipeline/job-crawler/sources.py` + `sources.yaml`: the free job APIs and their on/off switches,
  with each source's terms and limits summarized.
- `pipeline/job-crawler/hn.py`: the Hacker News "Who's Hiring" collector.
- `pipeline/job-crawler/locations.py` and `titles.py`: the location and title rules that `crawl.py`
  and `discover.py` share, so both agree on what counts as a match.
- `pipeline/job-crawler/http_util.py`: polite downloading (retries, a time budget, a curl fallback).
- `pipeline/job-crawler/liveness.py`: an optional ghost-job re-checker. Re-verify a posting is still
  live before you spend time on it.
- `pipeline/job-crawler/test_*.py`: offline tests (`test_crawl`, `test_discover`, `test_http_util`,
  `test_liveness`, `test_sources`); run them after any change, as `SETUP.md` shows.
- `pipeline/job-crawler/boards.yaml`: company boards. Starts empty; `discover.py` fills it, and you
  can add rows by hand.
- `pipeline/job-crawler/filters.yaml`: your role titles, keywords, and location rules. **Edit this:
  `include_titles` is the field that targets the whole crawler at your search.**
- `pipeline/job-crawler/README.md`: the crawler's detailed docs (every flag, every source, how to add
  a board or a source, the sponsorship-flag mechanism).

## This is a generic tool, not tuned to any one field

Nothing here assumes a particular role, industry, or visa/work-authorization situation. Put your own
titles in `filters.yaml` and the crawler and discovery both follow them. The file's comments walk
through the hard-wall vs soft-flag rules, including how one requirement (a license, say) can be a
dead end for one person and a plus for another (see `SETUP.md`).
