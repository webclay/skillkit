---
name: firecrawl
description: Firecrawl web scraping API for AI applications. Trigger words - firecrawl, scrape, scraping, crawl, extract data, web content, parse website, crawl docs
---

# Firecrawl Web Scraping

Web scraping API with JavaScript rendering, anti-bot handling, and LLM-ready output.

## When to Use This Skill

- Scraping websites with JavaScript content
- Extracting structured data from web pages
- Crawling documentation or entire sites
- Getting clean markdown for AI/LLM consumption
- Monitoring competitor prices or content

## When NOT to Use

- Simple static pages (use fetch directly)
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

## Initialize Client

```typescript
// lib/firecrawl.ts
import FirecrawlApp from '@mendable/firecrawl-js';

export const firecrawl = new FirecrawlApp({
  apiKey: process.env.FIRECRAWL_API_KEY!
});
```

## Scrape Single Page

```typescript
// Get clean markdown from any URL
const result = await firecrawl.scrapeUrl('https://example.com/article', {
  formats: ['markdown']
});

console.log(result.markdown); // Clean markdown content
console.log(result.metadata); // Title, description, etc.
```

## Extract Structured Data

```typescript
import { z } from 'zod';

const ArticleSchema = z.object({
  title: z.string(),
  author: z.string(),
  publishDate: z.string(),
  content: z.string(),
  tags: z.array(z.string())
});

const result = await firecrawl.scrapeUrl(url, {
  formats: ['extract'],
  extract: {
    schema: ArticleSchema.shape,
    systemPrompt: 'Extract article information from the page'
  }
});

const article = ArticleSchema.parse(result.extract);
```

## Crawl Multiple Pages

```typescript
// Crawl entire website
const crawlResult = await firecrawl.crawlUrl('https://docs.example.com', {
  limit: 100,
  includePaths: ['/docs/*'],
  excludePaths: ['/docs/archive/*'],
  scrapeOptions: {
    formats: ['markdown']
  }
});

// Process all pages
for (const page of crawlResult.data) {
  console.log(page.url, page.markdown);
}
```

## Advanced Options

```typescript
const result = await firecrawl.scrapeUrl(url, {
  formats: ['markdown'],
  // Wait for dynamic content
  waitFor: 5000,
  // Remove unwanted elements
  removeSelectors: ['nav', 'footer', '.advertisement', '#comments'],
  // Custom headers
  headers: {
    'Authorization': `Bearer ${token}`
  }
});
```

## RAG with AI

```typescript
import { openrouter } from '@/lib/openrouter';

async function answerFromWebsite(url: string, question: string) {
  // Scrape content
  const crawlResult = await firecrawl.crawlUrl(url, {
    limit: 50,
    scrapeOptions: { formats: ['markdown'] }
  });

  // Combine content
  const context = crawlResult.data
    .map(page => `URL: ${page.url}\n\n${page.markdown}`)
    .join('\n\n---\n\n');

  // Ask LLM
  const completion = await openrouter.chat.completions.create({
    model: 'anthropic/claude-3.5-sonnet',
    messages: [
      { role: 'system', content: 'Answer based on the website content.' },
      { role: 'user', content: `Context:\n${context}\n\nQuestion: ${question}` }
    ]
  });

  return completion.choices[0].message.content;
}
```

## Rate-Limited Batch Scraping

```typescript
import pLimit from 'p-limit';

async function batchScrape(urls: string[], concurrency = 3) {
  const limit = pLimit(concurrency);

  return Promise.all(
    urls.map(url =>
      limit(async () => {
        try {
          const result = await firecrawl.scrapeUrl(url, {
            formats: ['markdown']
          });
          return { url, success: true, markdown: result.markdown };
        } catch (error: any) {
          return { url, success: false, error: error.message };
        }
      })
    )
  );
}
```

## Caching Results

```typescript
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL!,
  token: process.env.UPSTASH_REDIS_REST_TOKEN!
});

async function scrapeCached(url: string, ttl = 3600) {
  const cacheKey = `scrape:${url}`;

  const cached = await redis.get<string>(cacheKey);
  if (cached) return cached;

  const result = await firecrawl.scrapeUrl(url, {
    formats: ['markdown']
  });

  await redis.setex(cacheKey, ttl, result.markdown || '');
  return result.markdown || '';
}
```

## Tips

- Use `formats: ['markdown']` for AI consumption
- Add `removeSelectors` to clean up navigation/ads
- Use `waitFor` for JavaScript-heavy pages
- Cache results to reduce API calls
- Respect robots.txt and rate limits

## How to Verify

### Quick Checks
- API key works: `await firecrawl.scrapeUrl('https://example.com')`
- Markdown is clean and readable
- Structured extraction matches schema

### Common Issues
- "Rate limit exceeded": Add delays between requests
- Empty markdown: Page may block scraping or need `waitFor`
- Missing content: Check if JavaScript renders after page load
