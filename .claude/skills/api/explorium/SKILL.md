---
name: explorium
description: Explorium API for B2B data enrichment, business intelligence, and prospect discovery. Trigger words - explorium, business enrichment, prospect enrichment, firmographics, b2b intelligence, company data, lead discovery, contact enrichment
---

# Explorium API

B2B data enrichment platform for business intelligence, prospect discovery, and contact information. Access 80M+ companies and comprehensive prospect data.

## When to Use This Skill

- Enriching company data with firmographics and technographics
- Discovering prospects at target companies
- Getting verified contact information (emails, phones)
- Building sales prospecting and lead generation tools
- Monitoring business events (funding, hiring, office changes)
- Market sizing and audience validation

## When NOT to Use

- Consumer/B2C data needs (Explorium is B2B focused)
- Real-time streaming data (use webhooks for events)
- Small-scale lookups under 500/month (free tier available)

## Setup

No SDK required - use fetch or axios directly.

### Environment Variables

```env
EXPLORIUM_API_KEY=your_api_key_here
```

Get your API key: [admin.explorium.ai](https://admin.explorium.ai) > Access & Authentication > Getting Your API Key

## Initialize Client

```typescript
// lib/explorium.ts
const EXPLORIUM_BASE_URL = "https://api.explorium.ai/v1";

async function exploriumFetch<T>(
  endpoint: string,
  options: RequestInit = {}
): Promise<T> {
  const response = await fetch(`${EXPLORIUM_BASE_URL}${endpoint}`, {
    ...options,
    headers: {
      "Content-Type": "application/json",
      "api_key": process.env.EXPLORIUM_API_KEY!,
      ...options.headers,
    },
  });

  if (!response.ok) {
    if (response.status === 429) {
      throw new Error("Rate limit exceeded - max 200 queries per minute");
    }
    throw new Error(`Explorium API error: ${response.status}`);
  }

  return response.json();
}
```

## Recommended Workflow

Explorium recommends a "broad to specific" approach:

1. **Stats** - Validate market size before fetching
2. **Fetch Businesses** - Get companies matching filters
3. **Fetch Prospects** - Find people at those companies
4. **Enrich Contacts** - Get emails and phone numbers

## Business Statistics

Check market size before committing to a full fetch.

```typescript
interface BusinessStatsParams {
  filters?: {
    country_code?: { values: string[] };
    company_size?: { values: string[] };
    industry?: { values: string[] };
  };
}

async function getBusinessStats(params: BusinessStatsParams) {
  return exploriumFetch("/businesses/stats", {
    method: "POST",
    body: JSON.stringify(params),
  });
}

// Usage - Check how many tech companies in US
const stats = await getBusinessStats({
  filters: {
    country_code: { values: ["US"] },
    industry: { values: ["Technology"] },
  },
});
```

## Fetch Businesses

Retrieve companies matching your criteria.

```typescript
interface FetchBusinessesParams {
  mode: "full" | "light";
  size?: number; // Max 10,000
  page?: number;
  page_size?: number; // Max 100
  filters?: {
    country_code?: { values: string[] };
    company_size?: { values: string[] };
    google_category?: { values: string[] };
    industry?: { values: string[] };
  };
}

interface Business {
  business_id: string;
  name: string;
  domain: string;
  industry: string;
  company_size: string;
  employee_count: number;
  country: string;
  city: string;
  founded_year: number;
  linkedin_url: string;
  website: string;
}

async function fetchBusinesses(
  params: FetchBusinessesParams
): Promise<{ data: Business[]; total: number }> {
  return exploriumFetch("/businesses", {
    method: "POST",
    body: JSON.stringify(params),
  });
}

// Usage
const businesses = await fetchBusinesses({
  mode: "full",
  page_size: 100,
  filters: {
    country_code: { values: ["US"] },
    company_size: { values: ["51-200", "201-500"] },
  },
});
```

## Fetch Prospects

Find people at target companies. Returns basic info without contact details.

```typescript
interface FetchProspectsParams {
  mode: "full" | "light";
  page?: number;
  page_size?: number; // Max 100
  filters?: {
    business_id?: { values: string[] };
    job_department?: { values: string[] };
    job_title?: { values: string[] };
    has_email?: boolean;
  };
}

interface Prospect {
  prospect_id: string;
  full_name: string;
  first_name: string;
  last_name: string;
  job_title: string;
  job_department: string;
  company_name: string;
  linkedin_url: string;
}

async function fetchProspects(
  params: FetchProspectsParams
): Promise<{ data: Prospect[]; total: number }> {
  return exploriumFetch("/prospects", {
    method: "POST",
    body: JSON.stringify(params),
  });
}

// Usage - Find sales leaders at specific companies
const prospects = await fetchProspects({
  mode: "full",
  page_size: 100,
  filters: {
    business_id: { values: ["8adce3ca1cef0c986b22310e369a0793"] },
    job_department: { values: ["Sales", "Marketing"] },
    has_email: true,
  },
});
```

## Enrich Prospect Contacts

Get verified emails and phone numbers for prospects.

```typescript
interface ContactInfo {
  prospect_id: string;
  emails: string[];
  professional_email: string;
  phone_numbers: string[];
  mobile_phone: string;
}

async function enrichProspectContacts(
  prospectId: string
): Promise<ContactInfo> {
  return exploriumFetch("/prospects/contacts_information/enrich", {
    method: "POST",
    body: JSON.stringify({ prospect_id: prospectId }),
  });
}

// Usage
const contact = await enrichProspectContacts("f0bc40c20b185d6b102662a6621632be");
console.log(contact.professional_email);
console.log(contact.mobile_phone);
```

## Bulk Business Enrichment

Enrich up to 50 businesses in a single request.

```typescript
async function bulkEnrichBusinesses(
  businessIds: string[]
): Promise<{ data: Business[] }> {
  if (businessIds.length > 50) {
    throw new Error("Maximum 50 business IDs per request");
  }

  return exploriumFetch("/businesses/bulk-enrich", {
    method: "POST",
    body: JSON.stringify({ business_ids: businessIds }),
  });
}

// Usage
const enriched = await bulkEnrichBusinesses([
  "8adce3ca1cef0c986b22310e369a0793",
  "a34bacf839b923770b2c360eefa26748",
]);
```

## Bulk Prospect Enrichment

Enrich up to 50 prospects in a single request.

```typescript
async function bulkEnrichProspects(
  prospectIds: string[]
): Promise<{ data: ContactInfo[] }> {
  if (prospectIds.length > 50) {
    throw new Error("Maximum 50 prospect IDs per request");
  }

  return exploriumFetch("/prospects/enrich/bulk", {
    method: "POST",
    body: JSON.stringify({ prospect_ids: prospectIds }),
  });
}
```

## Match Business by Domain/Name

Find the business_id for a known company.

```typescript
async function matchBusiness(params: {
  name?: string;
  domain?: string;
}): Promise<{ business_id: string; confidence: number }> {
  return exploriumFetch("/businesses/match", {
    method: "POST",
    body: JSON.stringify(params),
  });
}

// Usage
const match = await matchBusiness({ domain: "tesla.com" });
console.log(match.business_id);
```

## Rate Limits

- **200 queries per minute** (qpm)
- Per-second and per-minute rate limits enforced
- Standard HTTP error codes for exceeded limits

## Free Tier

- Enrich up to **500 records per month** for free
- Full API access on all subscription tiers

## Important Notes

- **Prospect fetch** returns basic info only - use contact enrichment for emails/phones
- **Match first** - Get business_id before enrichment for accuracy
- **Pagination** - Max 100 records per page
- **Bulk limits** - Max 50 IDs per bulk request
- **ID extraction** - Always extract business_id/prospect_id for downstream calls

## How to Verify

### Quick Checks

- API key works: Call `/businesses/stats` with simple filters
- Business fetch returns data with valid filters
- Prospect enrichment returns email/phone fields

### Common Issues

- 401 Unauthorized: Check api_key header (not API_KEY or x-api-key)
- 429 Rate Limited: Exceeded 200 qpm, add delays
- Empty results: Broaden filters or check data availability
- No contact info: Must call enrichment endpoint separately
