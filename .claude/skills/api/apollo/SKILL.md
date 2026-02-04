---
name: apollo
description: Apollo.io API for lead enrichment, people search, and sales automation. Trigger words - apollo, lead enrichment, prospect search, people enrichment, contact enrichment, b2b data, sales leads, sequences
---

# Apollo.io API

B2B sales intelligence API for lead enrichment, people search, contact management, and sequence automation.

## When to Use This Skill

- Enriching lead/prospect data with business information
- Searching for people by company, title, or demographics
- Building sales prospecting tools
- Automating outreach sequences
- Managing contacts and accounts programmatically

## When NOT to Use

- Consumer data (Apollo is B2B focused)
- Real-time data needs (use webhooks where available)
- Simple contact forms (use standard form handling)

## Setup

No SDK available - use fetch or axios directly.

### Environment Variables

```env
APOLLO_API_KEY=your_api_key_here
```

Get your API key: Apollo Dashboard > Settings > Integrations > API > Connect

## Initialize Client

```typescript
// lib/apollo.ts
const APOLLO_BASE_URL = "https://api.apollo.io/api/v1";

async function apolloFetch<T>(
  endpoint: string,
  options: RequestInit = {}
): Promise<T> {
  const response = await fetch(`${APOLLO_BASE_URL}${endpoint}`, {
    ...options,
    headers: {
      "Content-Type": "application/json",
      "x-api-key": process.env.APOLLO_API_KEY!,
      ...options.headers,
    },
  });

  if (!response.ok) {
    if (response.status === 429) {
      throw new Error("Rate limit exceeded - wait and retry");
    }
    throw new Error(`Apollo API error: ${response.status}`);
  }

  return response.json();
}
```

## People Enrichment

Enrich a single person's data by email, name, or LinkedIn URL.

```typescript
interface ApolloPersonEnrichment {
  person: {
    id: string;
    first_name: string;
    last_name: string;
    email: string;
    title: string;
    seniority: string;
    departments: string[];
    organization: {
      id: string;
      name: string;
      website_url: string;
      industry: string;
      estimated_num_employees: number;
    };
    employment_history: Array<{
      organization_name: string;
      title: string;
      start_date: string;
      end_date: string | null;
    }>;
  };
}

async function enrichPerson(params: {
  email?: string;
  first_name?: string;
  last_name?: string;
  organization_name?: string;
  domain?: string;
  linkedin_url?: string;
  reveal_personal_emails?: boolean;
  reveal_phone_number?: boolean;
}): Promise<ApolloPersonEnrichment> {
  return apolloFetch<ApolloPersonEnrichment>("/people/match", {
    method: "POST",
    body: JSON.stringify(params),
  });
}

// Usage
const result = await enrichPerson({
  email: "john@company.com",
  reveal_personal_emails: true,
});

console.log(result.person.title); // "VP of Engineering"
console.log(result.person.organization.name); // "Company Inc"
```

## Bulk People Enrichment

Enrich up to 10 people in a single API call.

```typescript
async function bulkEnrichPeople(
  details: Array<{
    email?: string;
    first_name?: string;
    last_name?: string;
    organization_name?: string;
    domain?: string;
  }>
): Promise<{ matches: ApolloPersonEnrichment["person"][] }> {
  return apolloFetch("/people/bulk_match", {
    method: "POST",
    body: JSON.stringify({ details }),
  });
}

// Usage
const results = await bulkEnrichPeople([
  { email: "john@company.com" },
  { first_name: "Jane", last_name: "Doe", domain: "startup.io" },
]);
```

## People Search

Search Apollo's database for prospects. Does not consume credits.

```typescript
interface PeopleSearchParams {
  person_titles?: string[];
  person_seniorities?: string[];
  organization_domains?: string[];
  organization_locations?: string[];
  organization_num_employees_ranges?: string[];
  page?: number;
  per_page?: number; // Max 100
}

interface PeopleSearchResult {
  people: Array<{
    id: string;
    first_name: string;
    last_name: string;
    title: string;
    organization: { name: string };
  }>;
  pagination: {
    page: number;
    per_page: number;
    total_entries: number;
    total_pages: number;
  };
}

async function searchPeople(
  params: PeopleSearchParams
): Promise<PeopleSearchResult> {
  return apolloFetch<PeopleSearchResult>("/mixed_people/api_search", {
    method: "POST",
    body: JSON.stringify(params),
  });
}

// Usage - Find VPs at tech companies
const prospects = await searchPeople({
  person_titles: ["VP", "Vice President"],
  person_seniorities: ["vp", "director"],
  organization_num_employees_ranges: ["51,200", "201,500"],
  per_page: 100,
});
```

## Create Contact

Add a contact to your Apollo database.

```typescript
interface CreateContactParams {
  first_name: string;
  last_name: string;
  email?: string;
  organization_name?: string;
  title?: string;
  phone_number?: string;
  run_dedupe?: boolean; // Prevent duplicates
}

async function createContact(params: CreateContactParams) {
  return apolloFetch("/contacts", {
    method: "POST",
    body: JSON.stringify(params),
  });
}

// Usage
await createContact({
  first_name: "John",
  last_name: "Smith",
  email: "john@company.com",
  organization_name: "Company Inc",
  title: "Director of Sales",
  run_dedupe: true, // Important!
});
```

## Add Contact to Sequence

Add existing contacts to an outreach sequence.

```typescript
async function addToSequence(
  sequenceId: string,
  contactIds: string[]
) {
  return apolloFetch(`/emailer_campaigns/${sequenceId}/add_contact_ids`, {
    method: "POST",
    body: JSON.stringify({ contact_ids: contactIds }),
  });
}

// Usage
await addToSequence("seq_abc123", ["contact_1", "contact_2"]);
```

## Search Sequences

Find sequences in your Apollo account.

```typescript
async function searchSequences(params?: { q?: string }) {
  return apolloFetch("/emailer_campaigns/search", {
    method: "POST",
    body: JSON.stringify(params || {}),
  });
}
```

## Rate Limits

Apollo uses fixed-window rate limiting per minute, hour, and day.

| Plan | Per Minute | Per Day |
|------|------------|---------|
| Free | 50 | 600 |
| Basic/Pro | 200 | 2,000 |
| Custom | Higher | Higher |

### Rate Limit Handling

```typescript
async function apolloFetchWithRetry<T>(
  endpoint: string,
  options: RequestInit = {},
  maxRetries = 3
): Promise<T> {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await apolloFetch<T>(endpoint, options);
    } catch (error: any) {
      if (error.message.includes("429") && attempt < maxRetries - 1) {
        // Wait 60 seconds for rate limit window to reset
        await new Promise((resolve) => setTimeout(resolve, 60000));
        continue;
      }
      throw error;
    }
  }
  throw new Error("Max retries exceeded");
}
```

## Important Notes

- **People Search** does not return emails/phones - use Enrichment endpoints after
- **Credits**: Enrichment consumes credits, search does not
- **Deduplication**: Always set `run_dedupe: true` when creating contacts
- **Master API Key**: Some endpoints require master key (sequences, usage stats)
- **Display Limit**: People search limited to 50,000 records (100 per page, 500 pages max)

## How to Verify

### Quick Checks

- API key works: Call `/people/match` with a known email
- Search returns results with valid filters
- Contacts appear in Apollo dashboard after creation

### Common Issues

- 401 Unauthorized: Check API key is correct
- 403 Forbidden: Endpoint requires master API key
- 429 Too Many Requests: Hit rate limit, wait 60 seconds
- Empty results: Add more specific filters or check data exists
- Duplicate contacts: Enable `run_dedupe: true`
