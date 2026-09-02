# SETUP

## 1. Install

```
pip install -r pipeline/job-crawler/requirements.txt
```
That's the one dependency (PyYAML). No API keys, no accounts, no login required for anything in
this repo.

## 2. Point it at your own search

Two files, both in `pipeline/job-crawler/`:

### `filters.yaml`

Edit `include_titles` to your own role titles: this is the field that actually targets the
crawler. A few worked examples for security-adjacent roles (delete and write your own):
```yaml
include_titles:
  - security engineer
  - application security
  - penetration tester
  - security researcher
  - incident response
  - soc analyst
  - threat intelligence
```

**🔴 One thing specific to your field, worth reading closely before you fill in
`exclude_patterns`:** the file ships with commented-out examples showing how someone WITHOUT a
security clearance might exclude clearance-required postings. **For you, the opposite may well be
true.** If you hold, or are eligible for, a security clearance, cleared/defense-adjacent roles are
often a premium slice of the security job market, not something to filter out. Consider instead
**adding** clearance-related terms to `include_keywords` so those roles rank higher, and leaving
`exclude_patterns` alone (or using it for something that's actually a dead end for you). This
repo has no opinion on which is right for you; it ships with both patterns unfilled specifically
so you make this call yourself rather than inheriting a stranger's default.

### `boards.yaml`

Add your target companies. Each entry needs the company name, which ATS they use
(Greenhouse/Lever/Ashby), and their board slug, found from their careers page URL:
```
boards.greenhouse.io/<slug>   -> ats: greenhouse
jobs.lever.co/<slug>          -> ats: lever
jobs.ashbyhq.com/<slug>       -> ats: ashby
```

## 3. Run it

```
python3 pipeline/job-crawler/crawl.py --hours 720   # wide window for a first run
```

You should get real, live postings back as JSON. If you get zero: widen `--hours`, check your
`include_titles` actually match how these companies phrase the role, or check `boards.yaml`'s
slugs (the JSON output's `errors` array names any that failed; a wrong slug is harmless, it's
just skipped).

Add `--write` if you also want a plain markdown table at `pipeline/job-feed.md` you can skim
without parsing JSON.

## 4. Before you apply to a lead, re-check it's still live

```
python3 pipeline/job-crawler/liveness.py --url <the posting URL>
```
Roughly one in five to seven postings are already dead by the time someone gets to them. This is
a quick, free, conservative check: it only ever says "dead" on strong evidence.

## Verify everything works

```
cd pipeline/job-crawler && python3 test_liveness.py -v
```
Should print `OK`. This is a fully offline, self-contained test suite with no network calls and
no dependency on your own config being filled in yet.

## Troubleshooting

- **`ModuleNotFoundError: No module named 'yaml'`** → `pip install PyYAML`.
- **0 matches** → see step 3 above.
- **A board's `errors` entry says "HTTP 404"** → the slug in `boards.yaml` is wrong; re-check the
  company's careers page URL.
