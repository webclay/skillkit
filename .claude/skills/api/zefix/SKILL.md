---
name: zefix
description: Use this skill when looking up Swiss company data via the Zefix Public REST API. Activate when the user mentions Zefix, Swiss company search, Handelsregister, UID/CHE numbers, commercial register lookups, or SOGC publications.
---

# Zefix Public REST API

Access to the Swiss Central Business Name Index (Zentraler Firmenindex). Look up Swiss companies by name, UID, or registry ID. Retrieve full company details including address, purpose, capital, legal form, and SOGC publications.

## When to Use This Skill

- User wants to search for Swiss companies
- User needs company details from the Swiss commercial register
- User mentions Zefix, Handelsregister, UID (CHE-xxx), or Swiss company lookup
- User wants to retrieve SOGC (Swiss Official Gazette of Commerce) publications
- User needs legal form, canton, or registry information for Swiss companies

## Instructions

### Step 1: Set Environment Variables

The API requires HTTP Basic Authentication with Zefix credentials.

```env
ZEFIX_USERNAME=your_username
ZEFIX_PASSWORD=your_password
```

Credentials are obtained by registering at the Zefix portal. The API is free but requires attribution (OGD Open use license).

**Endpoints:**
- Production: `https://www.zefix.admin.ch/ZefixPublicREST/api/v1`
- Test/Integration: `https://www.zefixintg.admin.ch/ZefixPublicREST/api/v1`

### Step 2: Create Zefix Client

**lib/zefix.ts:**
```ts
const ZEFIX_BASE_URL = process.env.ZEFIX_BASE_URL || 'https://www.zefix.admin.ch/ZefixPublicREST/api/v1';
const ZEFIX_USERNAME = process.env.ZEFIX_USERNAME!;
const ZEFIX_PASSWORD = process.env.ZEFIX_PASSWORD!;

function getAuthHeader(): string {
  return 'Basic ' + Buffer.from(`${ZEFIX_USERNAME}:${ZEFIX_PASSWORD}`).toString('base64');
}

async function zefixFetch<T>(path: string, options?: RequestInit): Promise<T> {
  const response = await fetch(`${ZEFIX_BASE_URL}${path}`, {
    ...options,
    headers: {
      'Content-Type': 'application/json; charset=utf-8',
      Authorization: getAuthHeader(),
      ...options?.headers,
    },
  });

  if (!response.ok) {
    const error = await response.json().catch(() => ({ error: { message: response.statusText } }));
    throw new Error(error.error?.message || `Zefix API error: ${response.status}`);
  }

  return response.json();
}
```

### Step 3: Add TypeScript Types

**lib/zefix-types.ts:**
```ts
// Multilingual text (German, French, Italian, English)
interface DFIEString {
  de: string;
  fr: string;
  it: string;
  en: string;
}

interface LegalForm {
  id: number;
  uid: string; // 4-char eCH-0097 code
  name: DFIEString;
  shortName: DFIEString;
}

interface Address {
  organisation: string | null;
  careOf: string | null;
  street: string | null;
  houseNumber: string | null;
  addon: string | null;
  poBox: string | null;
  city: string | null;
  swissZipCode: string | null;
}

type CompanyStatus = 'ACTIVE' | 'CANCELLED' | 'BEING_CANCELLED';

interface CompanyShort {
  name: string;
  ehraid: number;
  uid: string | null;
  chid: string | null;
  legalSeatId: number;
  legalSeat: string;
  registryOfCommerceId: number;
  legalForm: LegalForm;
  status: CompanyStatus | null;
  sogcDate: string | null;
  deletionDate: string | null;
}

interface CompanyFull extends CompanyShort {
  translation: string[] | null;
  purpose: string | null;
  address: Address;
  canton: string;
  capitalNominal: string | null;
  capitalCurrency: string | null;
  headOffices: CompanyShort[] | null;
  furtherHeadOffices: CompanyShort[] | null;
  branchOffices: CompanyShort[] | null;
  hasTakenOver: CompanyShort[] | null;
  wasTakenOverBy: CompanyShort[] | null;
  auditCompanies: CompanyShort[] | null;
  oldNames: CompanyOldName[] | null;
  sogcPub: SogcPublication[] | null;
  cantonalExcerptWeb: string;
  zefixDetailWeb: DFIEString;
}

interface CompanyOldName {
  name: string;
  sequenceNr: number;
  translation: string[] | null;
}

interface CompanySearchQuery {
  name: string; // min 3 chars, supports * wildcard
  legalFormId?: number;
  legalFormUid?: string; // 4-char eCH-0097 code
  registryOfCommerceId?: number;
  legalSeatId?: number;
  canton?: string; // 2-char abbreviation (e.g. "ZH", "BE")
  activeOnly?: boolean;
}

interface MutationType {
  id: number;
  key: string;
}

interface SogcPublication {
  sogcDate: string | null;
  sogcId: number | null;
  registryOfCommerceId: number | null;
  registryOfCommerceCanton: string | null;
  registryOfCommerceJournalId: number | null;
  registryOfCommerceJournalDate: string | null;
  message: string | null;
  mutationTypes: MutationType[] | null;
}

interface SogcPublicationAndCompanyShort {
  sogcPublication: SogcPublication;
  companyShort: CompanyShort;
}

interface RegistryOfCommerce {
  registryOfCommerceId: number;
  canton: string;
  address1: string; // Name of registry
  address2: string; // Street
  address3: string; // PO Box
  address4: string; // Zip + City
  homepage: string;
  url2: string; // Excerpt URL for active companies
  url3: string; // Contact email
  url4: string; // Excerpt URL for deleted companies
}

interface BfsCommunity {
  bfsId: number;
  canton: string;
  name: string;
  registryOfCommerceId: number;
}
```

### Step 4: Add API Methods

Add these to **lib/zefix.ts:**

```ts
// Search companies by name (min 3 chars, * wildcard supported)
export async function searchCompanies(query: CompanySearchQuery): Promise<CompanyShort[]> {
  return zefixFetch<CompanyShort[]>('/company/search', {
    method: 'POST',
    body: JSON.stringify(query),
  });
}

// Get full company details by UID (format: CHE-XXXXXXXXX)
export async function getCompanyByUid(uid: string): Promise<CompanyFull[]> {
  return zefixFetch<CompanyFull[]>(`/company/uid/${uid}`);
}

// Get full company details by EHRA-ID (internal federal registry ID)
export async function getCompanyByEhraid(ehraid: number): Promise<CompanyFull> {
  return zefixFetch<CompanyFull>(`/company/ehraid/${ehraid}`);
}

// Get full company details by CH-ID (legacy 13-digit number)
export async function getCompanyByChid(chid: string): Promise<CompanyFull[]> {
  return zefixFetch<CompanyFull[]>(`/company/chid/${chid}`);
}

// Get a specific SOGC publication by ID
export async function getSogcPublication(id: number): Promise<SogcPublicationAndCompanyShort> {
  return zefixFetch<SogcPublicationAndCompanyShort>(`/sogc/${id}`);
}

// Get all SOGC publications for a date (format: YYYY-MM-DD)
export async function getSogcPublicationsByDate(date: string): Promise<SogcPublicationAndCompanyShort[]> {
  return zefixFetch<SogcPublicationAndCompanyShort[]>(`/sogc/bydate/${date}`);
}

// List all cantonal registries of commerce
export async function getRegistriesOfCommerce(): Promise<RegistryOfCommerce[]> {
  return zefixFetch<RegistryOfCommerce[]>('/registryOfCommerce');
}

// Get registry by BFS community ID
export async function getRegistryByCommunityId(bfsId: string): Promise<RegistryOfCommerce> {
  return zefixFetch<RegistryOfCommerce>(`/registryOfCommerce/byBfsCommunityId/${bfsId}`);
}

// List all legal forms
export async function getLegalForms(): Promise<LegalForm[]> {
  return zefixFetch<LegalForm[]>('/legalForm');
}

// List all Swiss communities
export async function getCommunities(): Promise<BfsCommunity[]> {
  return zefixFetch<BfsCommunity[]>('/community');
}
```

## Examples

**Search for a company by name:**
```ts
import { searchCompanies } from '@/lib/zefix';

const results = await searchCompanies({
  name: 'Novartis',
  activeOnly: true,
});

// results: CompanyShort[] with name, UID, legal seat, legal form, status
```

**Search with wildcard:**
```ts
const results = await searchCompanies({
  name: 'Swiss*',
  canton: 'ZH',
  activeOnly: true,
});
```

**Get full company details by UID:**
```ts
import { getCompanyByUid } from '@/lib/zefix';

const companies = await getCompanyByUid('CHE-123.456.789');
const company = companies[0];

console.log({
  name: company.name,
  purpose: company.purpose,
  status: company.status,
  canton: company.canton,
  legalForm: company.legalForm.name.en,
  capital: `${company.capitalNominal} ${company.capitalCurrency}`,
  address: {
    street: company.address.street,
    city: company.address.city,
    zip: company.address.swissZipCode,
  },
  zefixLink: company.zefixDetailWeb.en,
});
```

**Get SOGC publications for a date:**
```ts
import { getSogcPublicationsByDate } from '@/lib/zefix';

const publications = await getSogcPublicationsByDate('2026-02-11');

publications.forEach((pub) => {
  console.log(`${pub.companyShort.name} (${pub.companyShort.uid})`);
  console.log(`  Published: ${pub.sogcPublication.sogcDate}`);
  console.log(`  Message: ${pub.sogcPublication.message}`);
});
```

**Filter by legal form and canton:**
```ts
// Get all active AG (Aktiengesellschaft) companies in Zurich
const results = await searchCompanies({
  name: '*',
  legalFormId: 3, // AG
  canton: 'ZH',
  activeOnly: true,
});
```

**Error handling:**
```ts
try {
  const companies = await getCompanyByUid('CHE-123.456.789');
  if (companies.length === 0) {
    console.log('Company not found');
    return;
  }
  // process company...
} catch (error) {
  if (error.message.includes('404')) {
    console.log('Company not found');
  } else if (error.message.includes('400')) {
    console.log('Invalid UID format');
  } else {
    console.error('Zefix API error:', error.message);
  }
}
```

## API Reference

**Base URL:** `https://www.zefix.admin.ch/ZefixPublicREST/api/v1`

**Auth:** HTTP Basic with Zefix credentials

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/company/search` | POST | Search companies by name (min 3 chars, * wildcard) |
| `/company/uid/{uid}` | GET | Get company by UID (CHE-XXXXXXXXX) |
| `/company/ehraid/{id}` | GET | Get company by EHRA-ID |
| `/company/chid/{id}` | GET | Get company by CH-ID (legacy) |
| `/sogc/{id}` | GET | Get SOGC publication by ID |
| `/sogc/bydate/{date}` | GET | Get SOGC publications by date (YYYY-MM-DD) |
| `/registryOfCommerce` | GET | List all cantonal registries |
| `/registryOfCommerce/byBfsCommunityId/{id}` | GET | Get registry by BFS community ID |
| `/legalForm` | GET | List all legal forms |
| `/community` | GET | List all Swiss communities |

**Error types:** `INTERNAL_SERVER_ERROR`, `INVALID_QUERY_WORDS`, `INVALID_REQUEST_DATA`, `RESULTLIST_TO_LARGE`, `NOT_FOUND`

**Status codes:** 200 (OK), 400 (Bad Request), 404 (Not Found), 500 (Server Error)

## Search Query Rules

- `name` is required, minimum 3 characters
- `*` wildcard supported (e.g. `Swiss*`, `*bank*`)
- `registryOfCommerceId`, `legalSeatId`, and `canton` are mutually exclusive - only use one at a time
- `legalFormId` and `legalFormUid` filter by legal form (use `/legalForm` endpoint to get valid values)
- `activeOnly: true` excludes cancelled and liquidating companies

## Common Legal Forms

| ID | UID Code | Name (EN) | Abbreviation |
|----|----------|-----------|--------------|
| 3 | 0106 | Company limited by shares | AG |
| 4 | 0107 | Limited liability company | GmbH |
| 9 | 0151 | Sole proprietorship | Einzelfirma |
| 6 | 0109 | Cooperative | Genossenschaft |
| 5 | 0108 | Association | Verein |
| 7 | 0110 | Foundation | Stiftung |

Use the `/legalForm` endpoint for the complete list.

## Swiss Cantons

Use 2-letter abbreviations: ZH, BE, LU, UR, SZ, OW, NW, GL, ZG, FR, SO, BS, BL, SH, AR, AI, SG, GR, AG, TG, TI, VD, VS, NE, GE, JU

## Tips

- UID format is `CHE-XXX.XXX.XXX` (with or without dots depending on context)
- The API returns multilingual data (DE/FR/IT/EN) for legal forms and Zefix links
- `CompanyFull` includes related companies (head offices, branches, audit companies, takeovers)
- SOGC publications contain the full gazette text in the `message` field
- Cache legal forms and communities locally - they rarely change
- The `cantonalExcerptWeb` field links directly to the cantonal commercial register entry

## How to Verify

### Quick Checks
- Search for "Novartis" returns results with valid UIDs
- Get company by known UID returns full details with address
- No authentication errors in terminal

### Manual Verification
- Compare results with Zefix web search: https://www.zefix.admin.ch/en/search/entity/welcome
- Verify UID numbers match the official format (CHE-XXX.XXX.XXX)
- Check that company status matches reality (active vs cancelled)

### Common Issues
- "401 Unauthorized": Check ZEFIX_USERNAME and ZEFIX_PASSWORD
- "400 Bad Request": Search name must be min 3 characters; check mutually exclusive filters
- "RESULTLIST_TO_LARGE": Search too broad - add filters (canton, legalForm, activeOnly)
- "NOT_FOUND": Company/UID doesn't exist in the registry
- Empty results: Try with `activeOnly: false` to include cancelled companies
