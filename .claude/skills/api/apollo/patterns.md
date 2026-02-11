# Apollo API Patterns

Advanced patterns and complete examples for Apollo.io integration.

## Apollo Service Class

```typescript
// lib/apollo-service.ts
import type { Person, Organization, Pagination } from '@/lib/apollo-types';

class ApolloService {
  private baseUrl = 'https://api.apollo.io/api/v1';
  private apiKey: string;

  constructor(apiKey: string) {
    this.apiKey = apiKey;
  }

  private async request<T>(endpoint: string, options: RequestInit = {}): Promise<T> {
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': this.apiKey,
        ...options.headers,
      },
    });

    if (response.status === 429) {
      const retryAfter = response.headers.get('Retry-After');
      throw new ApolloRateLimitError('Rate limit exceeded', parseInt(retryAfter || '60', 10));
    }

    if (!response.ok) {
      const error = await response.json().catch(() => ({}));
      throw new ApolloAPIError(error.message || `HTTP ${response.status}`, response.status);
    }

    return response.json();
  }

  async enrichPerson(params: Record<string, string | boolean>): Promise<{ person: Person }> {
    const query = new URLSearchParams();
    Object.entries(params).forEach(([k, v]) => { if (v !== undefined) query.set(k, String(v)); });
    return this.request(`/people/match?${query}`, { method: 'POST' });
  }

  async bulkEnrichPeople(details: Record<string, string>[]): Promise<{ matches: Person[]; credits_consumed: number }> {
    return this.request('/people/bulk_match', { method: 'POST', body: JSON.stringify({ details }) });
  }

  async searchPeople(params: Record<string, unknown>): Promise<{ people: Person[]; pagination: Pagination }> {
    return this.request('/mixed_people/api_search', { method: 'POST', body: JSON.stringify(params) });
  }

  async enrichOrganization(params: Record<string, string>): Promise<{ organization: Organization }> {
    const query = new URLSearchParams(params);
    return this.request(`/organizations/enrich?${query}`);
  }

  async bulkEnrichOrganizations(details: Record<string, string>[]): Promise<{ organizations: Organization[] }> {
    return this.request('/organizations/bulk_enrich', { method: 'POST', body: JSON.stringify({ details }) });
  }

  async searchOrganizations(params: Record<string, unknown>): Promise<{ organizations: Organization[]; pagination: Pagination }> {
    return this.request('/mixed_companies/search', { method: 'POST', body: JSON.stringify(params) });
  }

  async createContact(params: Record<string, unknown>): Promise<{ contact: Person }> {
    return this.request('/contacts', { method: 'POST', body: JSON.stringify(params) });
  }

  async getUsageStats(): Promise<Record<string, unknown>> {
    return this.request('/usage_stats', { method: 'GET' });
  }
}

class ApolloAPIError extends Error {
  constructor(message: string, public statusCode: number) {
    super(message);
    this.name = 'ApolloAPIError';
  }
}

class ApolloRateLimitError extends Error {
  constructor(message: string, public retryAfter: number) {
    super(message);
    this.name = 'ApolloRateLimitError';
  }
}

export const apollo = new ApolloService(process.env.APOLLO_API_KEY!);
```

## Batch Processing with Rate Limiting

```typescript
// lib/apollo-batch.ts
import { apollo } from './apollo-service';
import type { Person } from '@/lib/apollo-types';

export async function batchEnrichEmails(
  emails: string[],
  delayMs = 200
): Promise<{ success: Person[]; failed: { input: string; error: string }[] }> {
  const success: Person[] = [];
  const failed: { input: string; error: string }[] = [];

  // Process in chunks of 10 (bulk endpoint limit)
  const chunks = chunkArray(emails, 10);

  for (const chunk of chunks) {
    try {
      const result = await apollo.bulkEnrichPeople(chunk.map(email => ({ email })));
      success.push(...result.matches);
    } catch (error: any) {
      if (error.name === 'ApolloRateLimitError') {
        await delay(error.retryAfter * 1000);
        const result = await apollo.bulkEnrichPeople(chunk.map(email => ({ email })));
        success.push(...result.matches);
      } else {
        chunk.forEach(email => failed.push({ input: email, error: error.message }));
      }
    }
    await delay(delayMs);
  }

  return { success, failed };
}

function chunkArray<T>(array: T[], size: number): T[][] {
  const chunks: T[][] = [];
  for (let i = 0; i < array.length; i += size) {
    chunks.push(array.slice(i, i + size));
  }
  return chunks;
}

function delay(ms: number): Promise<void> {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

## Search with Async Generator

```typescript
// lib/apollo-search.ts
import { apollo } from './apollo-service';
import type { Person } from '@/lib/apollo-types';

export async function* searchAllPeople(params: {
  person_titles?: string[];
  person_seniorities?: string[];
  q_organization_domains?: string[];
  maxResults?: number;
}): AsyncGenerator<Person[], void, unknown> {
  const { maxResults = 1000, ...searchParams } = params;
  let page = 1;
  let totalFetched = 0;

  while (totalFetched < maxResults) {
    const result = await apollo.searchPeople({ ...searchParams, page, per_page: 100 });
    if (result.people.length === 0) break;

    yield result.people;

    totalFetched += result.people.length;
    page++;

    if (page > 500) break; // Apollo max: 50,000 results
    if (page >= result.pagination.total_pages) break;

    await new Promise(r => setTimeout(r, 100));
  }
}

// Usage
async function findAllVPs() {
  const allPeople: Person[] = [];

  for await (const batch of searchAllPeople({
    person_titles: ['VP', 'Vice President'],
    person_seniorities: ['vp'],
    maxResults: 500,
  })) {
    allPeople.push(...batch);
    console.log(`Fetched ${allPeople.length} people`);
  }

  return allPeople;
}
```

## Webhook Handler for Phone Reveal

```typescript
// app/api/webhooks/apollo/route.ts (Next.js)
// or equivalent server route for your framework
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const payload = await request.json();

  if (payload.type === 'phone_number_revealed') {
    const { person_id, phone_numbers } = payload.data;

    // Update your database with the phone numbers
    await db.lead.update({
      where: { apolloId: person_id },
      data: {
        phoneNumbers: phone_numbers.map((p: any) => p.sanitized_number),
      },
    });
  }

  return NextResponse.json({ received: true });
}
```

## Search Filter Reference

```typescript
// Valid seniority values
const SENIORITY_LEVELS = [
  'owner', 'founder', 'c_suite', 'partner', 'vp',
  'head', 'director', 'manager', 'senior', 'entry', 'intern',
] as const;

// Valid department values
const DEPARTMENTS = [
  'engineering', 'finance', 'human_resources', 'information_technology',
  'legal', 'marketing', 'operations', 'sales', 'support',
] as const;

// Valid employee count ranges
const EMPLOYEE_RANGES = [
  '1,10', '11,20', '21,50', '51,100', '101,200',
  '201,500', '501,1000', '1001,2000', '2001,5000',
  '5001,10000', '10001,',
] as const;
```
