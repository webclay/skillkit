---
name: firecrawl
description: Firecrawl web scraping, crawling, search, and file parsing for AI applications. Covers scrape (markdown, extract, question, highlights, video, audio, branding), crawl (multi-page with pagination), search (web, news, images with domain filtering), parse (local file upload - PDF, DOCX, XLSX), map (URL discovery), batch scrape, lockdown mode, browser sessions, and caching. Trigger words - firecrawl, web scraping, crawl website, extract data, parse pdf, parse document, scrape url, crawl docs, firecrawl search, batch scrape, lockdown mode
---

# Firecrawl Web Scraping

Web scraping API with JavaScript rendering, anti-bot handling, and LLM-ready output. Scrape pages, crawl sites, search the web, and parse local files - all returning clean structured data.

## Source of Truth

Official documentation: https://docs.firecrawl.dev/

Always check the official docs when the skill content conflicts or when you need the latest API details.

## When to Use This Skill

- Scraping websites with JavaScript content
- Extracting structured data from web pages with schemas
- Crawling documentation or entire sites
- Getting clean markdown for AI/LLM consumption
- Searching the web with domain filtering
- Parsing local files (PDF, DOCX, XLSX) into markdown
- Asking questions about page content
- Extracting highlights/quotes from pages
- Downloading video/audio from supported sites
- Extracting brand identity (colors, fonts, components) from websites

## When NOT to Use

- Simple static pages with no JS (use fetch directly)
- Scraping sites that prohibit it
- High-volume scraping without rate limiting

## Setup

```bash
npm install @mendable/firecrawl-js
```

### Environment Variables

```env
FIRECRAWL_API_KEY=fc-xxxxxxxxxxxxx
```

### Initialize Client

```typescript
import Firecrawl from '@mendable/firecrawl-js';

const firecrawl = new Firecrawl({
  apiKey: process.env.FIRECRAWL_API_KEY!,
});
```

If `FIRECRAWL_API_KEY` is set as an environment variable, the SDK picks it up automatically (no need to pass `apiKey`).

## Scrape Single Page

```typescript
const result = await firecrawl.scrape('https://example.com/article', {
  formats: ['markdown'],
});

console.log(result.markdown);
console.log(result.metadata);
```

### Available Formats

| Format | What It Returns | Extra Credits |
|--------|----------------|---------------|
| `markdown` | Clean markdown text | 0 |
| `html` | Cleaned HTML | 0 |
| `rawHtml` | Unmodified source HTML | 0 |
| `links` | All hyperlinks on the page | 0 |
| `images` | All image URLs | 0 |
| `summary` | Condensed page overview | 0 |
| `json` | Structured data via schema or prompt | +4 |
| `screenshot` | Page screenshot (URL expires in 24h) | 0 |
| `branding` | Colors, fonts, spacing, UI components | 0 |
| `question` | Answer to a natural-language question | +4 |
| `highlights` | Relevant source text matching a query | +4 |
| `audio` | MP3 download URL (YouTube, etc.) | +4 |
| `video` | Video download URL (YouTube, etc.) | +4 |

Base cost: 1 credit per scrape. Format add-ons stack.

## Extract Structured Data

```typescript
import { z } from 'zod';

const ArticleSchema = z.object({
  title: z.string(),
  author: z.string(),
  publishDate: z.string(),
  content: z.string(),
  tags: z.array(z.string()),
});

const result = await firecrawl.scrape(url, {
  formats: ['json'],
  extract: {
    schema: ArticleSchema.shape,
    systemPrompt: 'Extract article information from the page',
  },
});

const article = ArticleSchema.parse(result.extract);
```

Prompt-based extraction (no schema):

```typescript
const result = await firecrawl.scrape(url, {
  formats: ['json'],
  extract: {
    prompt: 'Extract the product name, price, and availability',
  },
});
```

## Question Format

Ask a natural-language question about a page. The answer comes back grounded in the page content.

```typescript
const result = await firecrawl.scrape('https://example.com/pricing', {
  formats: ['question'],
  question: 'What is the price of the Pro plan?',
});

console.log(result.answer);
```

Max 10,000 characters for the question. Costs 5 credits per page (1 base + 4). Works in both `/scrape` and `/search` endpoints.

## Highlights Format

Extract exact matching sentences, code blocks, or table rows from a page.

```typescript
const result = await firecrawl.scrape('https://docs.example.com/api', {
  formats: ['highlights'],
  query: 'rate limiting and authentication requirements',
});

console.log(result.highlights);
```

Consecutive matching sentences rejoin into paragraphs. Code lines wrap in fenced blocks. Table rows rebuild as markdown tables.

## Lockdown Mode

Restricts results to Firecrawl's index only - no outbound HTTP requests, no robots.txt calls, no media downloads.

```typescript
const result = await firecrawl.scrape('https://example.com', {
  formats: ['markdown'],
  lockdown: true,
});
```

Costs +4 credits when enabled. Cache misses cost 1 credit. Use when you need guaranteed isolation from external requests.

## Parse Local Files

Upload local files (PDF, DOCX, DOC, ODT, RTF, XLSX, XLS, HTML) up to 50 MB. Returns clean markdown, structured data, or summaries.

```typescript
import fs from 'node:fs';

const doc = await firecrawl.parse({
  data: fs.readFileSync('./report.pdf'),
  filename: 'report.pdf',
});

console.log(doc.markdown);
```

### HTML string input

```typescript
const doc = await firecrawl.parse(
  {
    data: '<html><body><h1>Hello</h1></body></html>',
    filename: 'upload.html',
    contentType: 'text/html',
  },
  { formats: ['markdown'] }
);
```

### PDF Parser Modes

| Mode | Use Case |
|------|----------|
| `fast` | Text extraction only, quickest |
| `auto` (default) | Text-first with OCR fallback for image-heavy pages |
| `ocr` | Full OCR, best for scanned documents |

Parse does NOT support: `changeTracking`, `screenshot`, `branding`, `actions`, `waitFor`, `location`, `mobile`.

## Crawl Multiple Pages

```typescript
const crawlResult = await firecrawl.crawl('https://docs.example.com', {
  limit: 100,
  includePaths: ['/docs/*'],
  excludePaths: ['/docs/archive/*'],
  scrapeOptions: {
    formats: ['markdown'],
  },
});

for (const page of crawlResult.data) {
  console.log(page.url, page.markdown);
}
```

### Async Crawl (for large sites)

```typescript
const { id } = await firecrawl.startCrawl('https://docs.example.com', {
  limit: 500,
});

const status = await firecrawl.getCrawlStatus(id);

await firecrawl.cancelCrawl(id);
```

### Real-Time Crawl Streaming

```typescript
const { id } = await firecrawl.startCrawl('https://example.com', { limit: 50 });
const watcher = firecrawl.watcher(id, { kind: 'crawl', pollInterval: 2 });

watcher.on('document', (doc) => console.log('Page:', doc.url));
watcher.on('done', (state) => console.log('Done:', state.status));

await watcher.start();
```

## Search the Web

```typescript
const results = await firecrawl.search('firecrawl pricing plans', {
  limit: 5,
  scrapeOptions: {
    formats: ['markdown'],
  },
});

for (const result of results.data) {
  console.log(result.url, result.title, result.markdown);
}
```

### Domain Filtering

```typescript
const results = await firecrawl.search('deployment guide', {
  includeDomains: ['docs.cloudflare.com', 'developers.cloudflare.com'],
  limit: 10,
});

const results2 = await firecrawl.search('javascript frameworks', {
  excludeDomains: ['w3schools.com', 'geeksforgeeks.org'],
  limit: 10,
});
```

`includeDomains` and `excludeDomains` are mutually exclusive. Pass domains without protocol or path.

### Search Sources

| Source | What It Searches |
|--------|-----------------|
| `web` (default) | General web results |
| `news` | News articles |
| `images` | Image results |

### Search Categories

| Category | What It Targets |
|----------|----------------|
| `github` | GitHub repos, code, issues |
| `research` | Academic papers (arXiv, Nature, IEEE, PubMed) |
| `pdf` | PDF documents |

### Time Filtering

```typescript
const results = await firecrawl.search('firecrawl updates', {
  tbs: 'qdr:w',  // Past week
});
```

| Filter | Period |
|--------|--------|
| `qdr:h` | Past hour |
| `qdr:d` | Past 24 hours |
| `qdr:w` | Past week |
| `qdr:m` | Past month |
| `qdr:y` | Past year |
| `sbd:1` | Sort by date (newest first) |

Credits: `ceil(results / 10) * 2` per search. Scraping adds per-result costs.

## Map (URL Discovery)

Discover all URLs on a website without scraping content.

```typescript
const res = await firecrawl.map('https://firecrawl.dev', { limit: 100 });
console.log(res.links);
```

## Batch Scrape

Process multiple URLs simultaneously.

```typescript
const { id } = await firecrawl.startBatchScrape(
  [
    'https://example.com/page1',
    'https://example.com/page2',
    'https://example.com/page3',
  ],
  { options: { formats: ['markdown'] } }
);

const status = await firecrawl.getBatchScrapeStatus(id);
```

Jobs expire after 24 hours.

## Advanced Scrape Options

```typescript
const result = await firecrawl.scrape(url, {
  formats: ['markdown'],
  waitFor: 5000,
  removeSelectors: ['nav', 'footer', '.advertisement', '#comments'],
  headers: {
    Authorization: `Bearer ${token}`,
  },
  location: {
    country: 'DE',
    languages: ['de', 'en'],
  },
});
```

### Caching

Default cache: 2 days. Control with:

```typescript
const result = await firecrawl.scrape(url, {
  formats: ['markdown'],
  maxAge: 0,          // Force fresh scrape (skip cache)
  storeInCache: false, // Don't cache this result
});
```

Cached results still cost 1 credit.

## Branding Extraction

Extract a site's full design system - colors, fonts, spacing, and UI components.

```typescript
const result = await firecrawl.scrape('https://example.com', {
  formats: ['branding'],
});

console.log(result.branding);
```

Returns: color scheme (light/dark), primary logo URL, color palette, font families, spacing defaults, border radius, button/input styles, animation settings, and brand personality traits.

## Credit Costs Reference

| Action | Credits |
|--------|---------|
| Scrape (base) | 1 |
| Cached result | 1 |
| JSON/extract format | +4 |
| Question format | +4 |
| Highlights format | +4 |
| Audio/video format | +4 |
| Enhanced proxy | +4 |
| Lockdown mode | +4 |
| Search | ceil(results/10) * 2 |
| PDF parse | 1 per page |
| Zero Data Retention | +1 |

## Tips

- Use `formats: ['markdown']` for AI consumption - it's the cheapest and most useful
- Add `removeSelectors` to clean up navigation, ads, and other noise
- Use `waitFor` for JavaScript-heavy pages that render content after load
- Cache results to reduce API calls (default 2-day cache)
- Use `lockdown: true` when you need guaranteed isolation from external requests
- Use `parse` for local files instead of scraping - no URL needed
- Use `question` format instead of scraping + LLM when you just need a specific answer
- Use `highlights` to pull exact quotes rather than processing full markdown yourself
- Use `search` with `includeDomains` to scope results to trusted sources
- Respect robots.txt and rate limits

## Common Gotchas

1. **`question` and `highlights` need their own parameters.** The format alone isn't enough - you must also pass `question: '...'` or `query: '...'` respectively.

2. **`includeDomains` and `excludeDomains` are mutually exclusive.** Pass one or the other in search, not both.

3. **Parse doesn't support all scrape options.** `changeTracking`, `screenshot`, `branding`, `actions`, `waitFor`, `location`, and `mobile` are not available on the parse endpoint.

4. **Batch scrape jobs expire after 24 hours.** Poll and save results before then.

5. **Screenshot URLs expire after 24 hours.** Download or cache them if you need them longer.

6. **Audio/video URLs expire after 1 hour.** Download immediately after scraping.

7. **`maxAge: 0` still costs 1 credit.** Fresh scrapes aren't free even if you disable caching.

8. **PDF parse costs per page, not per file.** A 50-page PDF costs 50 credits. Use the `maxPages` parser option to limit processing.

9. **Empty markdown usually means JS rendering.** Try adding `waitFor: 5000` or check if the site blocks scraping.

10. **Search credits scale with result count.** `ceil(results/10) * 2` - requesting 11 results costs 4 credits, not 2.

## How to Verify

### Quick Checks
- API key works: `await firecrawl.scrape('https://example.com')`
- Markdown is clean and readable
- Structured extraction matches schema

### Common Issues
- "Rate limit exceeded": Add delays between requests or use batch scrape
- Empty markdown: Page may block scraping or need `waitFor`
- Missing content: Check if JavaScript renders after page load
- Parse fails on large PDF: Increase `timeout` or limit `maxPages`
