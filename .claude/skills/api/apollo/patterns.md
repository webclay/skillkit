# Apollo API Patterns

Advanced patterns and complete examples for Apollo.io integration.

## Full TypeScript Types

```typescript
// types/apollo.ts

export interface ApolloPerson {
  id: string;
  first_name: string;
  last_name: string;
  name: string;
  email: string;
  email_status: "verified" | "unverified" | "invalid";
  title: string;
  seniority: string;
  departments: string[];
  photo_url: string | null;
  linkedin_url: string | null;
  organization_id: string;
  organization: ApolloOrganization;
  employment_history: ApolloEmployment[];
  phone_numbers: ApolloPhone[];
  personal_emails: string[];
  city: string;
  state: string;
  country: string;
}

export interface ApolloOrganization {
  id: string;
  name: string;
  website_url: string;
  linkedin_url: string | null;
  industry: string;
  estimated_num_employees: number;
  founded_year: number | null;
  logo_url: string | null;
  primary_domain: string;
  city: string;
  state: string;
  country: string;
}

export interface ApolloEmployment {
  organization_name: string;
  organization_id: string | null;
  title: string;
  start_date: string;
  end_date: string | null;
  current: boolean;
}

export interface ApolloPhone {
  raw_number: string;
  sanitized_number: string;
  type: "work" | "mobile" | "home";
  position: number;
}

export interface ApolloContact {
  id: string;
  first_name: string;
  last_name: string;
  email: string;
  title: string;
  organization_name: string;
  owner_id: string;
  created_at: string;
  updated_at: string;
}

export interface ApolloSequence {
  id: string;
  name: string;
  active: boolean;
  num_steps: number;
  created_at: string;
}

export interface ApolloPagination {
  page: number;
  per_page: number;
  total_entries: number;
  total_pages: number;
}
```

## Apollo Service Class

```typescript
// lib/apollo-service.ts
import type {
  ApolloPerson,
  ApolloContact,
  ApolloSequence,
  ApolloPagination,
} from "@/types/apollo";

class ApolloService {
  private baseUrl = "https://api.apollo.io/api/v1";
  private apiKey: string;

  constructor(apiKey: string) {
    this.apiKey = apiKey;
  }

  private async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      ...options,
      headers: {
        "Content-Type": "application/json",
        "x-api-key": this.apiKey,
        ...options.headers,
      },
    });

    if (response.status === 429) {
      const retryAfter = response.headers.get("Retry-After");
      throw new ApolloRateLimitError(
        "Rate limit exceeded",
        parseInt(retryAfter || "60", 10)
      );
    }

    if (!response.ok) {
      const error = await response.json().catch(() => ({}));
      throw new ApolloAPIError(
        error.message || `HTTP ${response.status}`,
        response.status
      );
    }

    return response.json();
  }

  // People Enrichment
  async enrichPerson(params: {
    email?: string;
    first_name?: string;
    last_name?: string;
    organization_name?: string;
    domain?: string;
    linkedin_url?: string;
    reveal_personal_emails?: boolean;
    reveal_phone_number?: boolean;
  }): Promise<{ person: ApolloPerson }> {
    return this.request("/people/match", {
      method: "POST",
      body: JSON.stringify(params),
    });
  }

  // Bulk Enrichment
  async bulkEnrichPeople(
    details: Array<{
      email?: string;
      first_name?: string;
      last_name?: string;
      domain?: string;
    }>
  ): Promise<{ matches: ApolloPerson[] }> {
    return this.request("/people/bulk_match", {
      method: "POST",
      body: JSON.stringify({ details }),
    });
  }

  // People Search
  async searchPeople(params: {
    person_titles?: string[];
    person_seniorities?: string[];
    organization_domains?: string[];
    organization_locations?: string[];
    organization_num_employees_ranges?: string[];
    q_keywords?: string;
    page?: number;
    per_page?: number;
  }): Promise<{
    people: ApolloPerson[];
    pagination: ApolloPagination;
  }> {
    return this.request("/mixed_people/api_search", {
      method: "POST",
      body: JSON.stringify(params),
    });
  }

  // Contact Management
  async createContact(params: {
    first_name: string;
    last_name: string;
    email?: string;
    organization_name?: string;
    title?: string;
    phone_number?: string;
    run_dedupe?: boolean;
  }): Promise<{ contact: ApolloContact }> {
    return this.request("/contacts", {
      method: "POST",
      body: JSON.stringify(params),
    });
  }

  async searchContacts(params: {
    q_keywords?: string;
    page?: number;
    per_page?: number;
  }): Promise<{
    contacts: ApolloContact[];
    pagination: ApolloPagination;
  }> {
    return this.request("/contacts/search", {
      method: "POST",
      body: JSON.stringify(params),
    });
  }

  // Sequences
  async searchSequences(params?: {
    q?: string;
  }): Promise<{ emailer_campaigns: ApolloSequence[] }> {
    return this.request("/emailer_campaigns/search", {
      method: "POST",
      body: JSON.stringify(params || {}),
    });
  }

  async addToSequence(
    sequenceId: string,
    contactIds: string[]
  ): Promise<{ contacts: ApolloContact[] }> {
    return this.request(`/emailer_campaigns/${sequenceId}/add_contact_ids`, {
      method: "POST",
      body: JSON.stringify({ contact_ids: contactIds }),
    });
  }

  // Usage Stats
  async getUsageStats(): Promise<{
    rate_limits: Record<string, {
      minute: { limit: number; remaining: number };
      hour: { limit: number; remaining: number };
      day: { limit: number; remaining: number };
    }>;
  }> {
    return this.request("/usage_stats", { method: "GET" });
  }
}

// Custom Errors
class ApolloAPIError extends Error {
  constructor(message: string, public statusCode: number) {
    super(message);
    this.name = "ApolloAPIError";
  }
}

class ApolloRateLimitError extends Error {
  constructor(message: string, public retryAfter: number) {
    super(message);
    this.name = "ApolloRateLimitError";
  }
}

// Export singleton
export const apollo = new ApolloService(process.env.APOLLO_API_KEY!);
```

## Batch Processing with Rate Limiting

```typescript
// lib/apollo-batch.ts
import { apollo } from "./apollo-service";

interface BatchResult<T> {
  success: T[];
  failed: Array<{ input: unknown; error: string }>;
}

export async function batchEnrichEmails(
  emails: string[],
  options: {
    concurrency?: number;
    delayMs?: number;
  } = {}
): Promise<BatchResult<ApolloPerson>> {
  const { concurrency = 5, delayMs = 200 } = options;
  const success: ApolloPerson[] = [];
  const failed: Array<{ input: unknown; error: string }> = [];

  // Process in chunks of 10 (bulk endpoint limit)
  const chunks = chunkArray(emails, 10);

  for (const chunk of chunks) {
    try {
      const result = await apollo.bulkEnrichPeople(
        chunk.map((email) => ({ email }))
      );
      success.push(...result.matches);
    } catch (error: any) {
      if (error.name === "ApolloRateLimitError") {
        // Wait and retry
        await delay(error.retryAfter * 1000);
        const result = await apollo.bulkEnrichPeople(
          chunk.map((email) => ({ email }))
        );
        success.push(...result.matches);
      } else {
        chunk.forEach((email) =>
          failed.push({ input: email, error: error.message })
        );
      }
    }

    // Respect rate limits
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
  return new Promise((resolve) => setTimeout(resolve, ms));
}
```

## Search with Pagination

```typescript
// lib/apollo-search.ts
import { apollo } from "./apollo-service";
import type { ApolloPerson } from "@/types/apollo";

export async function* searchAllPeople(params: {
  person_titles?: string[];
  person_seniorities?: string[];
  organization_domains?: string[];
  maxResults?: number;
}): AsyncGenerator<ApolloPerson[], void, unknown> {
  const { maxResults = 1000, ...searchParams } = params;
  let page = 1;
  let totalFetched = 0;

  while (totalFetched < maxResults) {
    const result = await apollo.searchPeople({
      ...searchParams,
      page,
      per_page: 100,
    });

    if (result.people.length === 0) break;

    yield result.people;

    totalFetched += result.people.length;
    page++;

    // Max 500 pages (50,000 results) - Apollo limit
    if (page > 500) break;
    if (page >= result.pagination.total_pages) break;

    // Rate limit delay
    await new Promise((r) => setTimeout(r, 100));
  }
}

// Usage
async function findAllVPs() {
  const allPeople: ApolloPerson[] = [];

  for await (const batch of searchAllPeople({
    person_titles: ["VP", "Vice President"],
    person_seniorities: ["vp"],
    maxResults: 500,
  })) {
    allPeople.push(...batch);
    console.log(`Fetched ${allPeople.length} people`);
  }

  return allPeople;
}
```

## Server Action Example (Next.js)

```typescript
// app/actions/apollo.ts
"use server";

import { apollo } from "@/lib/apollo-service";
import { z } from "zod";

const enrichSchema = z.object({
  email: z.string().email(),
});

export async function enrichLeadAction(formData: FormData) {
  const parsed = enrichSchema.safeParse({
    email: formData.get("email"),
  });

  if (!parsed.success) {
    return { error: "Invalid email address" };
  }

  try {
    const result = await apollo.enrichPerson({
      email: parsed.data.email,
      reveal_personal_emails: false,
    });

    return {
      success: true,
      person: {
        name: `${result.person.first_name} ${result.person.last_name}`,
        title: result.person.title,
        company: result.person.organization?.name,
        linkedin: result.person.linkedin_url,
      },
    };
  } catch (error: any) {
    if (error.name === "ApolloRateLimitError") {
      return { error: "Too many requests. Please try again later." };
    }
    return { error: "Failed to enrich lead" };
  }
}
```

## API Route Example

```typescript
// app/api/leads/enrich/route.ts
import { NextRequest, NextResponse } from "next/server";
import { apollo } from "@/lib/apollo-service";

export async function POST(request: NextRequest) {
  try {
    const { email } = await request.json();

    if (!email) {
      return NextResponse.json(
        { error: "Email is required" },
        { status: 400 }
      );
    }

    const result = await apollo.enrichPerson({ email });

    return NextResponse.json({
      person: result.person,
    });
  } catch (error: any) {
    if (error.name === "ApolloRateLimitError") {
      return NextResponse.json(
        { error: "Rate limit exceeded" },
        {
          status: 429,
          headers: { "Retry-After": String(error.retryAfter) },
        }
      );
    }

    return NextResponse.json(
      { error: "Internal server error" },
      { status: 500 }
    );
  }
}
```

## Webhook Handler for Phone Reveal

```typescript
// app/api/webhooks/apollo/route.ts
import { NextRequest, NextResponse } from "next/server";

export async function POST(request: NextRequest) {
  const payload = await request.json();

  // Apollo sends webhook when phone number is revealed
  if (payload.type === "phone_number_revealed") {
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

## Seniority and Department Filters

```typescript
// Valid seniority values for Apollo search
const SENIORITY_LEVELS = [
  "owner",
  "founder",
  "c_suite",
  "partner",
  "vp",
  "head",
  "director",
  "manager",
  "senior",
  "entry",
  "intern",
] as const;

// Valid department values
const DEPARTMENTS = [
  "engineering",
  "finance",
  "human_resources",
  "information_technology",
  "legal",
  "marketing",
  "operations",
  "sales",
  "support",
] as const;

// Example: Find engineering leaders
const engineeringLeaders = await apollo.searchPeople({
  person_seniorities: ["vp", "director", "head"],
  person_departments: ["engineering"],
  organization_num_employees_ranges: ["51,200"],
});
```

## Employee Count Range Formats

```typescript
// Valid employee count ranges for filtering
const EMPLOYEE_RANGES = [
  "1,10", // 1-10 employees
  "11,20", // 11-20 employees
  "21,50", // 21-50 employees
  "51,100", // 51-100 employees
  "101,200", // 101-200 employees
  "201,500", // 201-500 employees
  "501,1000", // 501-1,000 employees
  "1001,2000", // 1,001-2,000 employees
  "2001,5000", // 2,001-5,000 employees
  "5001,10000", // 5,001-10,000 employees
  "10001,", // 10,001+ employees
] as const;
```
