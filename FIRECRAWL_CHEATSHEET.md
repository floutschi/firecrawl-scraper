# Firecrawl MCP Cheat Sheet

Reference for the `mcp__firecrawl__*` tools available once the Firecrawl MCP server is connected (`claude mcp list` should show `firecrawl` as Connected). Use this whenever asked to pull data from a specific website.

## Which tool for which job

| Need | Tool |
|---|---|
| Content of one known URL | `firecrawl_scrape` |
| Screenshot of a page | `firecrawl_scrape` (`formats: ["screenshot"]`) |
| Don't know the exact page, just the site | `firecrawl_map` (list URLs) → `firecrawl_scrape` the ones you want |
| Content from many/all pages on a site | `firecrawl_crawl` |
| Open web search (not a specific site) | `firecrawl_search` |
| Multi-source synthesis / research report | `firecrawl_agent` + `firecrawl_agent_status` |
| Page needs clicks/login/forms/pagination first | `firecrawl_interact` |
| Local file (PDF, DOCX, XLSX...) | `firecrawl_parse` |
| Track a page/site for changes over time | `firecrawl_monitor_*` |
| Programming question (docs, issues, PRs) | `firecrawl_developer_search` |
| Academic/biomedical papers | `firecrawl_research_*` |

## Core tools

### `firecrawl_scrape` — one known URL
```
url: "https://example.com/page"
formats: ["markdown"]        # add "links" to get outbound links, "json" for structured extraction
```
- `formats: ["json"]` + `jsonOptions: { prompt, schema }` — extract structured fields instead of raw markdown.
- `formats: ["screenshot"]` + `screenshotOptions: { fullPage, viewport, quality }` — capture a screenshot instead of/alongside content.
- Other formats worth knowing exist: `html`, `rawHtml`, `summary`, `branding`, `changeTracking`, `audio` — see the tool schema when one of these fits better than markdown/json.
- `onlyMainContent: true` strips nav/footer boilerplate.
- `proxy: "stealth"` or `"enhanced"` (options are `basic`/`stealth`/`enhanced`/`auto`) — try this before `firecrawl_interact` if a scrape comes back blocked, empty, or clearly bot-walled.
- `maxAge` — Firecrawl may serve a cached copy; set `maxAge: 0` to force a live fetch.
- `waitFor` — ms to wait for JS-rendered content before capturing.
- **Gotcha**: a site's root/landing page (e.g. `jobs.omr.com`) is often a marketing page with only a few teaser items. Check the `links` output or look for an "All ..." / "View all" link and scrape the actual listing page instead (e.g. `omr.com/en/jobs-hr/jobs`).
- Listing pages are often paginated (e.g. a `Next` link to `/jobs/p/2`) — scrape follow-up pages individually if you need the full set.

### `firecrawl_map` — discover URLs on a site (no content fetched)
```
url: "https://example.com"
search: "pricing"     # optional filter term
limit: 100
```
Use before `scrape`/`crawl` when you don't know the exact page path.

### `firecrawl_crawl` — pull many pages from one site
```
url: "https://example.com/docs"
limit: 50
includePaths: ["/docs/*"]
scrapeOptions: { formats: ["markdown"] }
```
Polls to completion and returns collected page data directly (also check with `firecrawl_check_crawl_status` if needed by ID). Keep `limit`/`maxDiscoveryDepth` conservative — full-site crawls can be large and slow.

### `firecrawl_search` — open web search, not tied to one site
```
query: "best CRM tools for startups"
sources: [{ type: "web" }]     # or "news" / "images"
limit: 5
scrapeOptions: { formats: ["markdown"] }   # optional: pull full content of top hits
```
Supports operators: quoted phrases, `-term`, `site:host`, `inurl:`, `intitle:`, `related:host`. `categories: ["developer"]` routes into the dev index (issues/PRs/READMEs/docs); `categories: ["research"]` restricts to research-affiliated sites (different from `firecrawl_research_*`, which searches paper abstracts directly).

### `firecrawl_agent` + `firecrawl_agent_status` — async multi-source research
```
firecrawl_agent: prompt: "...", urls: [optional seed URLs], schema: {optional}
→ returns a job id
firecrawl_agent_status: id: "<job id>"
→ poll every 15-30s until status is "completed" or "failed"
```
Use for synthesis across multiple sources when the task can wait (can take minutes). For anything time-sensitive, fall back to `firecrawl_search` + `firecrawl_scrape` instead.

### `firecrawl_interact` — live browser session (clicks, forms, login, pagination)
```
url: "https://example.com" (or scrapeId to reuse a session)
prompt: "click 'Load more' twice, then extract all product names"
# or: code + language: "bash" | "python" | "node"
```
This acts on the **live** site — form submissions etc. are real side effects. Call `firecrawl_interact_stop` with the `scrapeId` when done to free the session.

### `firecrawl_parse` — local files (PDF, DOCX, DOC, ODT, RTF, XLSX, XLS, HTML)
```
filePath: "C:\path\to\file.pdf"
formats: ["markdown"]
pdfOptions: { maxPages: 50 }
```
Not for URLs — use `firecrawl_scrape` for anything on the web.

## Monitoring (recurring checks)
- `firecrawl_monitor_create` — simple form: `page`/`pages` (or `queries` for a search monitor) + `goal` (plain-language description of what counts as a meaningful change) + `scheduleText`.
- `firecrawl_monitor_list` / `firecrawl_monitor_get` — list/inspect monitors.
- `firecrawl_monitor_run` — trigger an immediate out-of-schedule check.
- `firecrawl_monitor_checks` / `firecrawl_monitor_check` — list past checks / read one check's page-level diffs (`same`/`new`/`changed`/`removed`/`error`).
- `firecrawl_monitor_update` / `firecrawl_monitor_delete` — edit or permanently remove a monitor.

Prefer this over repeated one-off scrapes when the same URL needs checking more than once (e.g. "let me know if this pricing page changes").

## Research index (academic papers)
Separate from `firecrawl_search`'s `categories: ["research"]` (which just filters web results to research-affiliated sites). These query Firecrawl's own paper index — mostly biomedical/life-science (PubMed, bioRxiv, medRxiv) plus arXiv.
- `firecrawl_research_search_papers` — semantic search over abstracts, with author/category/date filters.
- `firecrawl_research_read_paper` — in-body passages relevant to a specific question (full text, indexed papers only).
- `firecrawl_research_inspect_paper` — canonical metadata for one paper ID (arXiv/PMC/PMID/DOI).
- `firecrawl_research_related_papers` — citation-graph expansion from 1-10 seed papers (`similar`/`citers`/`references`).
- `firecrawl_research_search_github` — indexed GitHub issues/PRs/READMEs.

## Feedback tools
- `firecrawl_feedback` (search/scrape/parse/map) and `firecrawl_search_feedback` (search only, by `searchId`) — optional quality feedback, not needed for normal usage. Skip unless explicitly asked to rate a result.

## Practical tips
- Default to `formats: ["markdown"]`; add `"links"` whenever you might need to follow up with more pages (pagination, related pages, "view all" links).
- For structured data (job listings, prices, tables) prefer `formats: ["json"]` with a `jsonOptions.schema` over parsing raw markdown yourself.
- If a page looks JS-heavy / content is missing, try a larger `waitFor` before reaching for `firecrawl_interact`.
- Check `metadata.statusCode` and `cacheState` in scrape results — `cacheState: "hit"` means it may be a cached copy; use `maxAge: 0` if freshness matters.
- Every call consumes Firecrawl credits (see `metadata.creditsUsed`) — keep `crawl`/`map` `limit`/`maxDiscoveryDepth` conservative, especially on large or unfamiliar sites.
