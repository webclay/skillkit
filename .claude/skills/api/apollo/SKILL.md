---
name: apollo
description: Apollo.io API for people and company data enrichment, search, and CRM operations. Trigger words - apollo, company enrichment, people enrichment, company data, people search, organization search, lead enrichment, sales intelligence, prospect, contact enrichment, company lookup, person lookup
---

# Apollo.io API

Sales intelligence platform for enriching people and company data, searching prospects, and managing contacts. Access company information, employee details, email addresses, phone numbers, and organization data.

## When to Use This Skill

- User wants to enrich company/organization data (revenue, employees, industry, funding)
- User wants to enrich people data (title, email, phone, employment history)
- User needs to search for companies or people matching specific criteria
- User mentions Apollo, lead enrichment, prospect enrichment, or sales intelligence
- User wants to look up company details by domain
- package.json contains Apollo-related packages

## When NOT to Use

- Consumer data (Apollo is B2B focused)
- Simple contact forms (use standard form handling)

## Instructions

### Step 1: Set Environment Variables

```env
APOLLO_API_KEY=your_api_key_here
```

Get your API key from Apollo Settings > Integrations > API Keys. All Apollo plans include basic API access. No SDK required - use fetch directly.

### Step 2: Create Apollo Client

**lib/apollo.ts:**
```ts
const APOLLO_BASE_URL = 'https://api.apollo.io/api/v1';
const APOLLO_API_KEY = process.env.APOLLO_API_KEY!;

async function apolloFetch<T>(path: string, options?: RequestInit): Promise<T> {
  const response = await fetch(`${APOLLO_BASE_URL}${path}`, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': APOLLO_API_KEY,
      ...options?.headers,
    },
  });

  if (!response.ok) {
    const error = await response.json().catch(() => ({ message: response.statusText }));
    throw new Error(error.message || `Apollo API error: ${response.status}`);
  }

  return response.json();
}
```

### Step 3: Add TypeScript Types

**lib/apollo-types.ts:**
```ts
interface PersonOrganization {
  id: string;
  name: string;
  website_url: string | null;
  primary_domain: string | null;
  industry: string | null;
  estimated_num_employees: number | null;
  founded_year: number | null;
  linkedin_url: string | null;
}

interface EmploymentHistory {
  organization_name: string;
  title: string;
  current: boolean;
  start_date: string | null;
  end_date: string | null;
}

interface Person {
  id: string;
  first_name: string;
  last_name: string;
  name: string;
  email: string | null;
  email_status: string | null; // 'verified' | 'guessed' | 'unavailable' | 'bounced'
  title: string | null;
  headline: string | null;
  linkedin_url: string | null;
  photo_url: string | null;
  city: string | null;
  state: string | null;
  country: string | null;
  organization_id: string | null;
  organization: PersonOrganization | null;
  employment_history: EmploymentHistory[];
  departments: string[];
  subdepartments: string[];
  seniority: string | null;
  is_likely_to_engage: boolean;
}

interface Organization {
  id: string;
  name: string;
  website_url: string | null;
  linkedin_url: string | null;
  twitter_url: string | null;
  facebook_url: string | null;
  primary_domain: string | null;
  industry: string | null;
  estimated_num_employees: number | null;
  annual_revenue: number | null;
  founded_year: number | null;
  city: string | null;
  state: string | null;
  country: string | null;
  street_address: string | null;
  postal_code: string | null;
  total_funding: number | null;
  latest_funding_stage: string | null;
  keywords: string[];
  current_technologies: { uid: string; name: string; category: string }[];
}

interface Pagination {
  page: number;
  per_page: number;
  total_entries: number;
  total_pages: number;
}
```

### Step 4: Add API Methods

Add these to **lib/apollo.ts:**

```ts
// --- People ---

// Enrich a single person by name, email, domain, or LinkedIn URL
export async function enrichPerson(params: {
  first_name?: string;
  last_name?: string;
  name?: string;
  email?: string;
  hashed_email?: string;
  organization_name?: string;
  domain?: string;
  id?: string;
  linkedin_url?: string;
  reveal_personal_emails?: boolean;
  reveal_phone_number?: boolean;
  webhook_url?: string;
}): Promise<{ person: Person }> {
  const query = new URLSearchParams();
  Object.entries(params).forEach(([key, value]) => {
    if (value !== undefined) query.set(key, String(value));
  });

  return apolloFetch<{ person: Person }>(`/people/match?${query.toString()}`, {
    method: 'POST',
  });
}

// Bulk enrich up to 10 people at once
export async function bulkEnrichPeople(
  details: Array<{
    email?: string;
    first_name?: string;
    last_name?: string;
    organization_name?: string;
    domain?: string;
    linkedin_url?: string;
  }>,
  options?: {
    reveal_personal_emails?: boolean;
    reveal_phone_number?: boolean;
    webhook_url?: string;
  }
): Promise<{ matches: Person[]; credits_consumed: number }> {
  const query = new URLSearchParams();
  if (options?.reveal_personal_emails) query.set('reveal_personal_emails', 'true');
  if (options?.reveal_phone_number) query.set('reveal_phone_number', 'true');
  if (options?.webhook_url) query.set('webhook_url', options.webhook_url);

  const qs = query.toString();
  return apolloFetch(`/people/bulk_match${qs ? `?${qs}` : ''}`, {
    method: 'POST',
    body: JSON.stringify({ details }),
  });
}

// Search for people (free - does not consume credits)
export async function searchPeople(
  filters: {
    person_titles?: string[];
    person_seniorities?: string[];
    person_departments?: string[];
    person_locations?: string[];
    q_organization_domains?: string[];
    organization_industry_tag_ids?: string[];
    organization_num_employees_ranges?: string[];
    person_name?: string;
  },
  page?: number,
  perPage?: number
): Promise<{ people: Person[]; pagination: Pagination }> {
  return apolloFetch(`/mixed_people/api_search`, {
    method: 'POST',
    body: JSON.stringify({ ...filters, page: page || 1, per_page: perPage || 25 }),
  });
}

// --- Organizations ---

// Enrich an organization by domain, name, or Apollo ID
export async function enrichOrganization(params: {
  domain?: string;
  organization_name?: string;
  id?: string;
}): Promise<{ organization: Organization }> {
  const query = new URLSearchParams();
  Object.entries(params).forEach(([key, value]) => {
    if (value !== undefined) query.set(key, value);
  });

  return apolloFetch<{ organization: Organization }>(`/organizations/enrich?${query.toString()}`);
}

// Bulk enrich up to 10 organizations at once
export async function bulkEnrichOrganizations(
  details: Array<{ domain?: string; name?: string; organization_name?: string }>
): Promise<{ organizations: Organization[] }> {
  return apolloFetch(`/organizations/bulk_enrich`, {
    method: 'POST',
    body: JSON.stringify({ details }),
  });
}

// Search for organizations
export async function searchOrganizations(
  filters: {
    organization_domains?: string[];
    organization_locations?: string[];
    organization_industry_tag_ids?: string[];
    organization_num_employees_ranges?: string[];
    q_organization_keyword_tags?: string[];
    q_organization_name?: string;
  },
  page?: number,
  perPage?: number
): Promise<{ organizations: Organization[]; pagination: Pagination }> {
  return apolloFetch(`/mixed_companies/search`, {
    method: 'POST',
    body: JSON.stringify({ ...filters, page: page || 1, per_page: perPage || 25 }),
  });
}

// --- Contacts ---

// Create a contact in Apollo CRM
export async function createContact(params: {
  first_name?: string;
  last_name?: string;
  email?: string;
  phone_number?: string;
  organization_id?: string;
  organization_name?: string;
  title?: string;
  contact_stage_id?: string;
  owner_id?: string;
  run_dedupe?: boolean;
}): Promise<{ contact: Person }> {
  return apolloFetch(`/contacts`, {
    method: 'POST',
    body: JSON.stringify(params),
  });
}
```

## Examples

**Enrich a person by email:**
```ts
import { enrichPerson } from '@/lib/apollo';

const { person } = await enrichPerson({
  email: 'tim@apollo.io',
  domain: 'apollo.io',
});

console.log({
  name: person.name,
  title: person.title,
  company: person.organization?.name,
  industry: person.organization?.industry,
  employees: person.organization?.estimated_num_employees,
});
```

**Enrich a person by LinkedIn URL:**
```ts
const { person } = await enrichPerson({
  linkedin_url: 'https://www.linkedin.com/in/tim-zheng-677ba010',
});
```

**Enrich a person by name + company:**
```ts
const { person } = await enrichPerson({
  first_name: 'Tim',
  last_name: 'Zheng',
  domain: 'apollo.io',
});
```

**Bulk enrich multiple people (max 10):**
```ts
import { bulkEnrichPeople } from '@/lib/apollo';

const { matches, credits_consumed } = await bulkEnrichPeople([
  { first_name: 'Tim', last_name: 'Zheng', domain: 'apollo.io' },
  { email: 'jane@example.com' },
  { linkedin_url: 'https://www.linkedin.com/in/johndoe' },
]);

console.log(`Enriched ${matches.length} people, used ${credits_consumed} credits`);
```

**Enrich a company by domain:**
```ts
import { enrichOrganization } from '@/lib/apollo';

const { organization } = await enrichOrganization({ domain: 'apollo.io' });

console.log({
  name: organization.name,
  industry: organization.industry,
  employees: organization.estimated_num_employees,
  revenue: organization.annual_revenue,
  founded: organization.founded_year,
  funding: organization.total_funding,
  tech: organization.current_technologies?.map(t => t.name),
});
```

**Bulk enrich multiple companies (max 10):**
```ts
import { bulkEnrichOrganizations } from '@/lib/apollo';

const { organizations } = await bulkEnrichOrganizations([
  { domain: 'apollo.io' },
  { domain: 'google.com' },
  { domain: 'stripe.com' },
]);
```

**Search for people (free - no credits):**
```ts
import { searchPeople } from '@/lib/apollo';

const { people, pagination } = await searchPeople({
  person_titles: ['CEO', 'CTO', 'Founder'],
  q_organization_domains: ['apollo.io'],
  person_locations: ['San Francisco, California, United States'],
});

people.forEach(person => {
  console.log(`${person.name} - ${person.title} at ${person.organization?.name}`);
});
```

**Search for organizations:**
```ts
import { searchOrganizations } from '@/lib/apollo';

const { organizations } = await searchOrganizations({
  q_organization_name: 'Apollo',
  organization_num_employees_ranges: ['101,500'],
  organization_locations: ['United States'],
});
```

**Create a CRM contact:**
```ts
import { createContact } from '@/lib/apollo';

const { contact } = await createContact({
  first_name: 'Jane',
  last_name: 'Doe',
  email: 'jane@example.com',
  organization_name: 'Acme Inc',
  title: 'VP of Engineering',
  run_dedupe: true, // Important: prevent duplicates
});
```

**Error handling:**
```ts
try {
  const { person } = await enrichPerson({ email: 'user@example.com' });
  if (!person) {
    console.log('Person not found');
    return;
  }
  // process person...
} catch (error) {
  if (error.message.includes('401')) {
    console.log('Invalid API key');
  } else if (error.message.includes('429')) {
    console.log('Rate limited - wait and retry');
  } else {
    console.error('Apollo API error:', error.message);
  }
}
```

**Paginate through search results:**
```ts
async function getAllPeople(filters: Parameters<typeof searchPeople>[0]) {
  const allPeople: Person[] = [];
  let page = 1;

  while (page <= 500) { // Apollo limit: 50,000 records (100/page x 500 pages)
    const { people, pagination } = await searchPeople(filters, page, 100);
    allPeople.push(...people);

    if (people.length < 100 || allPeople.length >= pagination.total_entries) break;
    page++;
  }

  return allPeople;
}
```

## API Reference

**Base URL:** `https://api.apollo.io/api/v1`

**Auth:** `x-api-key: YOUR_API_KEY` header (or `Authorization: Bearer TOKEN` for OAuth)

| Endpoint | Method | Description | Credits |
|----------|--------|-------------|---------|
| `/people/match` | POST | Enrich single person | Yes |
| `/people/bulk_match` | POST | Enrich up to 10 people | Yes |
| `/organizations/enrich` | GET | Enrich single organization | Yes |
| `/organizations/bulk_enrich` | POST | Enrich up to 10 organizations | Yes |
| `/mixed_people/api_search` | POST | Search people | Free |
| `/mixed_companies/search` | POST | Search organizations | Yes |
| `/contacts` | POST | Create CRM contact | No |

**Status codes:** 200 (OK), 400 (Bad Request), 401 (Unauthorized), 422 (Unprocessable), 429 (Rate Limited)

## Rate Limits

- **Strategy:** Fixed-window rate limiting (per minute, per hour, per day)
- **Limits:** Plan-dependent - check Apollo dashboard or call `/usage_stats`
- **Bulk endpoints:** 50% of single endpoint's per-minute rate; 100% hourly/daily
- **Search display limit:** 50,000 records (100 per page, max 500 pages)
- **429 response:** Includes a message with the current limit

## Credit System

- **People Search is free** - does not consume credits
- Enrichment endpoints consume credits per matched record
- Bulk requests consume credits equal to successfully matched records
- Waterfall enrichment (email/phone from third-party sources) costs additional credits
- Credit amounts depend on your Apollo pricing plan
- Track usage: Apollo Settings > API Keys

## Search Filter Values

**Seniority levels:** `owner`, `founder`, `c_suite`, `partner`, `vp`, `head`, `director`, `manager`, `senior`, `entry`, `intern`

**Departments:** `engineering`, `finance`, `human_resources`, `information_technology`, `legal`, `marketing`, `operations`, `sales`, `support`

**Employee count ranges:** `1,10`, `11,20`, `21,50`, `51,100`, `101,200`, `201,500`, `501,1000`, `1001,2000`, `2001,5000`, `5001,10000`, `10001,`

## Tips

- Provide multiple data points for better matching (name + domain is better than name alone)
- `reveal_personal_emails` and `reveal_phone_number` require explicit opt-in (default: false)
- Phone number reveal requires a `webhook_url` for async delivery via webhook
- GDPR regions may restrict personal email access
- Always set `run_dedupe: true` when creating contacts to prevent duplicates
- `email_status` indicates: `verified`, `guessed`, `unavailable`, or `bounced`
- `is_likely_to_engage` indicates prospect engagement likelihood
- Master API key required for some endpoints (sequences, usage stats)

## How to Verify

### Quick Checks
- Enrich a known person returns valid data with organization info
- Search returns results with pagination metadata
- No authentication errors in terminal

### Manual Verification
- Compare enriched data with LinkedIn profiles
- Verify company data against company websites
- Check that email_status values make sense

### Common Issues
- "401 Unauthorized": Check APOLLO_API_KEY is valid
- "429 Too Many Requests": Rate limited - implement exponential backoff
- "422 Unprocessable": Invalid parameters - check request body format
- Empty results: Provide more identifying data (name + domain, not name alone)
- No emails returned: Set `reveal_personal_emails: true` and check credit balance
- Phone not returning: Ensure `webhook_url` is HTTPS and publicly accessible
- Duplicate contacts: Always enable `run_dedupe: true`
