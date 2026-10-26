# Scraper

This project is for pulling data **from the web**: scraping pages, extracting content, taking screenshots, crawling sites, and mapping site structure. If the task involves a website, it belongs here.

## Tooling

Use the **Firecrawl MCP server** (`mcp__firecrawl__*` tools) for any website task — scraping, extraction, screenshots, crawling, mapping, search, or monitoring. Don't use `WebFetch`/`WebSearch` for this project; Firecrawl is the dedicated tool and handles JS-rendered pages, structured extraction, and bulk operations that the built-ins can't.

Before using Firecrawl, read **[FIRECRAWL_CHEATSHEET.md](FIRECRAWL_CHEATSHEET.md)** — it maps each task type (single page, unknown page, whole site, open search, research, interactive/login-gated pages, local files, recurring monitoring) to the right tool and the key parameters for it. Check it every time rather than guessing which tool fits.

## Secrets

`FIRECRAWL_API_KEY` lives in `.env` (gitignored) for any scripts that call the Firecrawl API directly outside of MCP. The MCP connection itself authenticates via OAuth, not this key.
