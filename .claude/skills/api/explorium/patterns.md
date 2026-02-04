# Explorium API Patterns

Advanced patterns and complete examples for Explorium integration.

## Full TypeScript Types

```typescript
// types/explorium.ts

export interface ExploriumBusiness {
  business_id: string;
  name: string;
  domain: string;
  website: string;
  industry: string;
  google_category: string;
  company_size: string;
  employee_count: number;
  revenue_range: string;
  founded_year: number;
  country: string;
  country_code: string;
  state: string;
  city: string;
  address: string;
  postal_code: string;
  phone: string;
  linkedin_url: string;
  facebook_url: string;
  twitter_url: string;
  technologies: string[];
  keywords: string[];
}

export interface ExploriumProspect {
  prospect_id: string;
  full_name: string;
  first_name: string;
  last_name: string;
  job_title: string;
  job_department: string;
  job_level: string;
  company_name: string;
  business_id: string;
  linkedin_url: string;
  location: string;
  country: string;
}

export interface ExploriumContactInfo {
  prospect_id: string;
  emails: string[];
  professional_email: string;
  personal_email: string;
  phone_numbers: string[];
  mobile_phone: string;
  work_phone: string;
  email_status: "verified" | "unverified" | "invalid";
}

export interface ExploriumPagination {
  page: number;
  page_size: number;
  total: number;
  total_pages: number;
}

export interface ExploriumBusinessStats {
  total_count: number;
  by_country: Record<string, number>;
  by_industry: Record<string, number>;
  by_company_size: Record<string, number>;
}

// Filter types
export interface ValueFilter {
  values: string[];
}

export interface RangeFilter {
  min?: number;
  max?: number;
}

export interface BusinessFilters {
  country_code?: ValueFilter;
  company_size?: ValueFilter;
  industry?: ValueFilter;
  google_category?: ValueFilter;
  employee_count?: RangeFilter;
  revenue_range?: ValueFilter;
  founded_year?: RangeFilter;
  technologies?: ValueFilter;
}

export interface ProspectFilters {
  business_id?: ValueFilter;
  job_department?: ValueFilter;
  job_title?: ValueFilter;
  job_level?: ValueFilter;
  has_email?: boolean;
  country?: ValueFilter;
}
```

## Explorium Service Class

```typescript
// lib/explorium-service.ts
import type {
  ExploriumBusiness,
  ExploriumProspect,
  ExploriumContactInfo,
  ExploriumBusinessStats,
  BusinessFilters,
  ProspectFilters,
} from "@/types/explorium";

class ExploriumService {
  private baseUrl = "https://api.explorium.ai/v1";
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
        "api_key": this.apiKey,
        ...options.headers,
      },
    });

    if (response.status === 429) {
      throw new ExploriumRateLimitError("Rate limit exceeded (200 qpm)");
    }

    if (!response.ok) {
      const error = await response.json().catch(() => ({}));
      throw new ExploriumAPIError(
        error.message || `HTTP ${response.status}`,
        response.status
      );
    }

    return response.json();
  }

  // Business Statistics
  async getBusinessStats(filters?: BusinessFilters): Promise<ExploriumBusinessStats> {
    return this.request("/businesses/stats", {
      method: "POST",
      body: JSON.stringify({ filters }),
    });
  }

  // Fetch Businesses
  async fetchBusinesses(params: {
    mode?: "full" | "light";
    size?: number;
    page?: number;
    page_size?: number;
    filters?: BusinessFilters;
  }): Promise<{ data: ExploriumBusiness[]; total: number }> {
    return this.request("/businesses", {
      method: "POST",
      body: JSON.stringify({
        mode: params.mode || "full",
        ...params,
      }),
    });
  }

  // Match Business
  async matchBusiness(params: {
    name?: string;
    domain?: string;
  }): Promise<{ business_id: string; confidence: number }> {
    return this.request("/businesses/match", {
      method: "POST",
      body: JSON.stringify(params),
    });
  }

  // Bulk Enrich Businesses
  async bulkEnrichBusinesses(
    businessIds: string[]
  ): Promise<{ data: ExploriumBusiness[] }> {
    if (businessIds.length > 50) {
      throw new Error("Maximum 50 business IDs per request");
    }
    return this.request("/businesses/bulk-enrich", {
      method: "POST",
      body: JSON.stringify({ business_ids: businessIds }),
    });
  }

  // Fetch Prospects
  async fetchProspects(params: {
    mode?: "full" | "light";
    page?: number;
    page_size?: number;
    filters?: ProspectFilters;
  }): Promise<{ data: ExploriumProspect[]; total: number }> {
    return this.request("/prospects", {
      method: "POST",
      body: JSON.stringify({
        mode: params.mode || "full",
        ...params,
      }),
    });
  }

  // Match Prospect
  async matchProspect(params: {
    first_name?: string;
    last_name?: string;
    full_name?: string;
    company_name?: string;
    linkedin_url?: string;
  }): Promise<{ prospect_id: string; confidence: number }> {
    return this.request("/prospects/match", {
      method: "POST",
      body: JSON.stringify(params),
    });
  }

  // Enrich Prospect Contact Info
  async enrichProspectContact(prospectId: string): Promise<ExploriumContactInfo> {
    return this.request("/prospects/contacts_information/enrich", {
      method: "POST",
      body: JSON.stringify({ prospect_id: prospectId }),
    });
  }

  // Bulk Enrich Prospects
  async bulkEnrichProspects(
    prospectIds: string[]
  ): Promise<{ data: ExploriumContactInfo[] }> {
    if (prospectIds.length > 50) {
      throw new Error("Maximum 50 prospect IDs per request");
    }
    return this.request("/prospects/enrich/bulk", {
      method: "POST",
      body: JSON.stringify({ prospect_ids: prospectIds }),
    });
  }
}

// Custom Errors
class ExploriumAPIError extends Error {
  constructor(message: string, public statusCode: number) {
    super(message);
    this.name = "ExploriumAPIError";
  }
}

class ExploriumRateLimitError extends Error {
  constructor(message: string) {
    super(message);
    this.name = "ExploriumRateLimitError";
  }
}

// Export singleton
export const explorium = new ExploriumService(process.env.EXPLORIUM_API_KEY!);
```

## Complete SDR Prospecting Workflow

```typescript
// lib/explorium-prospecting.ts
import { explorium } from "./explorium-service";
import type {
  ExploriumBusiness,
  ExploriumProspect,
  ExploriumContactInfo,
} from "@/types/explorium";

interface ProspectingResult {
  business: ExploriumBusiness;
  prospects: Array<ExploriumProspect & { contact?: ExploriumContactInfo }>;
}

export async function runProspectingWorkflow(params: {
  targetIndustry: string;
  targetCountry: string;
  companySizes: string[];
  jobDepartments: string[];
  maxCompanies?: number;
  maxProspectsPerCompany?: number;
}): Promise<ProspectingResult[]> {
  const {
    targetIndustry,
    targetCountry,
    companySizes,
    jobDepartments,
    maxCompanies = 10,
    maxProspectsPerCompany = 5,
  } = params;

  const results: ProspectingResult[] = [];

  // Step 1: Check market size
  const stats = await explorium.getBusinessStats({
    country_code: { values: [targetCountry] },
    industry: { values: [targetIndustry] },
    company_size: { values: companySizes },
  });

  console.log(`Found ${stats.total_count} matching companies`);

  if (stats.total_count === 0) {
    return [];
  }

  // Step 2: Fetch businesses
  const businesses = await explorium.fetchBusinesses({
    mode: "full",
    page_size: Math.min(maxCompanies, 100),
    filters: {
      country_code: { values: [targetCountry] },
      industry: { values: [targetIndustry] },
      company_size: { values: companySizes },
    },
  });

  // Step 3 & 4: For each business, fetch and enrich prospects
  for (const business of businesses.data.slice(0, maxCompanies)) {
    const prospects = await explorium.fetchProspects({
      mode: "full",
      page_size: maxProspectsPerCompany,
      filters: {
        business_id: { values: [business.business_id] },
        job_department: { values: jobDepartments },
        has_email: true,
      },
    });

    // Enrich contacts in bulk
    const prospectIds = prospects.data.map((p) => p.prospect_id);
    const contacts = prospectIds.length > 0
      ? await explorium.bulkEnrichProspects(prospectIds)
      : { data: [] };

    // Merge prospect and contact data
    const enrichedProspects = prospects.data.map((prospect) => ({
      ...prospect,
      contact: contacts.data.find((c) => c.prospect_id === prospect.prospect_id),
    }));

    results.push({
      business,
      prospects: enrichedProspects,
    });

    // Rate limit protection - 200 qpm max
    await delay(300);
  }

  return results;
}

function delay(ms: number): Promise<void> {
  return new Promise((resolve) => setTimeout(resolve, ms));
}
```

## Paginated Business Search

```typescript
// lib/explorium-search.ts
import { explorium } from "./explorium-service";
import type { ExploriumBusiness, BusinessFilters } from "@/types/explorium";

export async function* searchAllBusinesses(
  filters: BusinessFilters,
  maxResults = 1000
): AsyncGenerator<ExploriumBusiness[], void, unknown> {
  let page = 1;
  let totalFetched = 0;
  const pageSize = 100;

  while (totalFetched < maxResults) {
    const result = await explorium.fetchBusinesses({
      mode: "full",
      page,
      page_size: pageSize,
      filters,
    });

    if (result.data.length === 0) break;

    yield result.data;

    totalFetched += result.data.length;
    page++;

    if (totalFetched >= result.total) break;

    // Rate limit protection
    await new Promise((r) => setTimeout(r, 300));
  }
}

// Usage
async function findAllTechCompanies() {
  const allBusinesses: ExploriumBusiness[] = [];

  for await (const batch of searchAllBusinesses(
    {
      country_code: { values: ["US"] },
      industry: { values: ["Technology"] },
      company_size: { values: ["51-200", "201-500"] },
    },
    500
  )) {
    allBusinesses.push(...batch);
    console.log(`Fetched ${allBusinesses.length} companies`);
  }

  return allBusinesses;
}
```

## Batch Processing with Rate Limiting

```typescript
// lib/explorium-batch.ts
import { explorium } from "./explorium-service";
import type { ExploriumContactInfo } from "@/types/explorium";

interface BatchResult<T> {
  success: T[];
  failed: Array<{ input: string; error: string }>;
}

export async function batchEnrichProspects(
  prospectIds: string[],
  options: { delayMs?: number } = {}
): Promise<BatchResult<ExploriumContactInfo>> {
  const { delayMs = 300 } = options;
  const success: ExploriumContactInfo[] = [];
  const failed: Array<{ input: string; error: string }> = [];

  // Process in chunks of 50 (bulk endpoint limit)
  const chunks = chunkArray(prospectIds, 50);

  for (const chunk of chunks) {
    try {
      const result = await explorium.bulkEnrichProspects(chunk);
      success.push(...result.data);
    } catch (error: any) {
      if (error.name === "ExploriumRateLimitError") {
        // Wait 60 seconds and retry
        await delay(60000);
        try {
          const result = await explorium.bulkEnrichProspects(chunk);
          success.push(...result.data);
        } catch (retryError: any) {
          chunk.forEach((id) =>
            failed.push({ input: id, error: retryError.message })
          );
        }
      } else {
        chunk.forEach((id) =>
          failed.push({ input: id, error: error.message })
        );
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
  return new Promise((resolve) => setTimeout(resolve, ms));
}
```

## Server Action Example (Next.js)

```typescript
// app/actions/explorium.ts
"use server";

import { explorium } from "@/lib/explorium-service";
import { z } from "zod";

const searchSchema = z.object({
  domain: z.string().min(1),
});

export async function enrichCompanyAction(formData: FormData) {
  const parsed = searchSchema.safeParse({
    domain: formData.get("domain"),
  });

  if (!parsed.success) {
    return { error: "Invalid domain" };
  }

  try {
    // Match the business first
    const match = await explorium.matchBusiness({
      domain: parsed.data.domain,
    });

    if (!match.business_id) {
      return { error: "Company not found" };
    }

    // Enrich with full data
    const enriched = await explorium.bulkEnrichBusinesses([match.business_id]);
    const business = enriched.data[0];

    return {
      success: true,
      company: {
        name: business.name,
        industry: business.industry,
        size: business.company_size,
        employees: business.employee_count,
        location: `${business.city}, ${business.country}`,
        website: business.website,
        linkedin: business.linkedin_url,
      },
    };
  } catch (error: any) {
    if (error.name === "ExploriumRateLimitError") {
      return { error: "Too many requests. Please try again later." };
    }
    return { error: "Failed to enrich company" };
  }
}
```

## API Route Example

```typescript
// app/api/prospects/search/route.ts
import { NextRequest, NextResponse } from "next/server";
import { explorium } from "@/lib/explorium-service";

export async function POST(request: NextRequest) {
  try {
    const { businessId, departments } = await request.json();

    if (!businessId) {
      return NextResponse.json(
        { error: "businessId is required" },
        { status: 400 }
      );
    }

    // Fetch prospects
    const prospects = await explorium.fetchProspects({
      mode: "full",
      page_size: 50,
      filters: {
        business_id: { values: [businessId] },
        job_department: departments ? { values: departments } : undefined,
        has_email: true,
      },
    });

    // Enrich contacts
    const prospectIds = prospects.data.map((p) => p.prospect_id);
    const contacts = prospectIds.length > 0
      ? await explorium.bulkEnrichProspects(prospectIds)
      : { data: [] };

    // Merge data
    const enrichedProspects = prospects.data.map((prospect) => ({
      ...prospect,
      contact: contacts.data.find((c) => c.prospect_id === prospect.prospect_id),
    }));

    return NextResponse.json({
      prospects: enrichedProspects,
      total: prospects.total,
    });
  } catch (error: any) {
    if (error.name === "ExploriumRateLimitError") {
      return NextResponse.json(
        { error: "Rate limit exceeded" },
        { status: 429 }
      );
    }

    return NextResponse.json(
      { error: "Internal server error" },
      { status: 500 }
    );
  }
}
```

## Common Filter Values

### Company Size

```typescript
const COMPANY_SIZES = [
  "1-10",
  "11-50",
  "51-200",
  "201-500",
  "501-1000",
  "1001-5000",
  "5001-10000",
  "10000+",
] as const;
```

### Job Departments

```typescript
const JOB_DEPARTMENTS = [
  "Sales",
  "Marketing",
  "Engineering",
  "Finance",
  "Human Resources",
  "Operations",
  "Legal",
  "IT",
  "Customer Success",
  "Product",
  "Executive",
] as const;
```

### Job Levels

```typescript
const JOB_LEVELS = [
  "C-Suite",
  "VP",
  "Director",
  "Manager",
  "Senior",
  "Entry",
  "Intern",
] as const;
```

## Webhook Configuration (Events)

```typescript
// For monitoring business/prospect events
// Configure webhooks at admin.explorium.ai

interface ExploriumWebhookPayload {
  event_type:
    | "funding_round"
    | "office_opening"
    | "office_closing"
    | "hiring_surge"
    | "layoffs"
    | "job_change"
    | "promotion";
  entity_type: "business" | "prospect";
  entity_id: string;
  data: Record<string, unknown>;
  timestamp: string;
}

// app/api/webhooks/explorium/route.ts
export async function POST(request: NextRequest) {
  const payload: ExploriumWebhookPayload = await request.json();

  switch (payload.event_type) {
    case "funding_round":
      // Company just raised funding
      await notifySalesTeam(payload.entity_id, payload.data);
      break;
    case "job_change":
      // Contact changed jobs
      await updateCRMContact(payload.entity_id, payload.data);
      break;
  }

  return NextResponse.json({ received: true });
}
```
