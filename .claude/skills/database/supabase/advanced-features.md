# Advanced Features

## PrivateLink (Enterprise)

For compliance requirements (healthcare, finance, public sector) or to minimize attack surface, Supabase offers PrivateLink - private database connections through AWS networks.

### What It Does

- Database traffic never leaves private AWS networks
- Your Supabase database appears to exist within your own VPC
- Lower latency than public connections (more direct routing)
- Can disable public database access entirely once configured

### Requirements

| Requirement | Details |
|-------------|---------|
| Plan | Team or Enterprise only |
| Cloud | AWS environments only |
| Region | Same AWS region as your Supabase project |
| Scope | Database connections only (not API, Auth, Storage, or Realtime) |

### When to Use

- Compliance requires private network connectivity
- Want to eliminate public database endpoints
- Workloads already running on AWS
- Need to meet SOC 2, HIPAA, or similar requirements

### When NOT to Use

- Free or Pro plan (not available)
- Non-AWS hosting (Vercel serverless, Cloudflare, etc.)
- Need private access to Auth, Storage, or Realtime (not supported yet)

### Setup Overview

1. **Add AWS Account ID** - In Supabase Dashboard > Project Settings > Infrastructure
2. **Accept Resource Share** - In AWS RAM (Resource Access Manager)
3. **Create VPC Endpoint** - In AWS VPC console, create endpoint for the Supabase service
4. **Configure Security Groups** - Allow TCP port 5432 inbound
5. **Update Connection String** - Use the private endpoint hostname
6. **Test Connection** - Verify connectivity from your VPC

### Connection String

After setup, your connection string changes from:
```
postgresql://postgres.[ref]:[password]@aws-0-[region].pooler.supabase.com:5432/postgres
```

To the private endpoint:
```
postgresql://postgres.[ref]:[password]@vpce-xxx.supabase.com:5432/postgres
```

### Limitations

- **Database only** - API calls, Auth, Storage, and Realtime still use public endpoints
- **Same region** - Your VPC must be in the same AWS region as your Supabase project
- **AWS only** - Not available for GCP, Azure, or other cloud providers

For most projects, the standard public connection with SSL is sufficient. PrivateLink is for organizations with strict compliance requirements.

## Pre-Request Hooks

Run custom validation before every Data API request. Useful for rate limiting, API key validation, or blocking access to specific tables.

### Setup

```sql
-- Create a function that runs before every request
CREATE OR REPLACE FUNCTION public.check_request()
RETURNS void AS $$
DECLARE
  req_path text := current_setting('request.path', true);
  req_headers json := current_setting('request.headers', true)::json;
BEGIN
  -- Example: Require custom API key header
  IF req_headers->>'x-api-key' IS NULL THEN
    RAISE EXCEPTION 'API key required'
      USING HINT = 'Include x-api-key header';
  END IF;

  -- Example: Block access to admin tables from API
  IF req_path LIKE '%admin_%' THEN
    RAISE EXCEPTION 'Access denied'
      USING HINT = 'Admin tables not accessible via API';
  END IF;
END;
$$ LANGUAGE plpgsql;

-- Enable the pre-request hook
ALTER ROLE authenticator SET pgrst.db_pre_request = 'public.check_request';
```

### Available Request Context

Inside your function, access request info with `current_setting()`:

| Setting | What It Contains |
|---------|------------------|
| `request.path` | API path (e.g., `/rest/v1/users`) |
| `request.headers` | JSON object of all headers |
| `request.method` | HTTP method (GET, POST, etc.) |
| `request.jwt.claims` | JWT payload (user info) |

### Use Cases

| Use Case | Implementation |
|----------|----------------|
| Rate limiting | Check counter in a rate_limits table |
| API key validation | Verify header against api_keys table |
| Block direct table access | Check path and reject certain patterns |
| Payment/quota checks | Query user's subscription status |
| IP allowlisting | Check request.headers->>'x-forwarded-for' |

### Limitations

- **Data API only** - Does not apply to Realtime or Storage
- For Realtime/Storage, add validation inside RLS policies instead

### When to Use

Most apps don't need pre-request hooks. Use them when:
- You need rate limiting at the database level
- You want to validate custom API keys
- You need to block certain tables from direct API access
- RLS alone isn't sufficient for your security requirements

## Security Advisor

Supabase includes a built-in Security Advisor that scans your project for misconfigurations.

**Access it:** Dashboard > Project Settings > Security Advisor

**What it checks:**
- Tables without RLS enabled
- Overly permissive RLS policies
- Exposed sensitive columns
- Missing indexes on foreign keys
- Other common security issues

**Run before production** - The Security Advisor catches issues that are easy to miss during development.

**AI Policy Assistant:** The dashboard also includes an AI assistant that can generate RLS policies from plain-language descriptions. Useful for complex policies.
