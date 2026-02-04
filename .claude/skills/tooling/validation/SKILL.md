---
name: validation
description: TypeScript validation libraries (Zod, Valibot, ArkType). Trigger words - zod, valibot, arktype, validation, schema, parse, validate input, form validation, env validation
---

# TypeScript Validation

Schema validation with compile-time type inference.

## When to Use This Skill

- API input validation
- Form validation
- Environment variable parsing
- Data transformation
- Type-safe parsing of external data

## Library Comparison

| Library | Best For | Bundle Size | Speed |
|---------|----------|-------------|-------|
| **Zod** | Ecosystem, familiarity | ~12kb | Fast |
| **Valibot** | Bundle size, tree-shaking | ~1-5kb | Fast |
| **ArkType** | Performance, complex types | ~25kb | Fastest |

All implement **Standard Schema** - interchangeable in tRPC, TanStack Form, etc.

## Zod (Recommended Default)

```bash
npm install zod
```

```typescript
import { z } from 'zod';

const userSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
  age: z.number().int().positive().optional()
});

type User = z.infer<typeof userSchema>;

const result = userSchema.safeParse(data);
if (result.success) {
  console.log(result.data); // typed as User
} else {
  console.log(result.error.flatten().fieldErrors);
}
```

## Valibot (Smallest Bundle)

```bash
npm install valibot
```

```typescript
import * as v from 'valibot';

const userSchema = v.object({
  name: v.pipe(v.string(), v.minLength(2)),
  email: v.pipe(v.string(), v.email()),
  age: v.optional(v.pipe(v.number(), v.integer(), v.minValue(1)))
});

type User = v.InferOutput<typeof userSchema>;

const result = v.safeParse(userSchema, data);
if (result.success) {
  console.log(result.output);
}
```

## ArkType (Fastest)

```bash
npm install arktype
```

```typescript
import { type } from 'arktype';

const user = type({
  name: 'string >= 2',
  email: 'email',
  'age?': 'integer > 0'
});

type User = typeof user.infer;

const result = user(data);
if (result instanceof type.errors) {
  console.log(result.summary);
} else {
  console.log(result); // typed as User
}
```

## Common Patterns

### Environment Variables

```typescript
// Zod
const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  API_KEY: z.string().min(1),
  PORT: z.coerce.number().default(3000)
});

export const env = envSchema.parse(process.env);
```

### API Input Validation

```typescript
const createUserInput = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  name: z.string().min(2).max(100)
});

export async function POST(req: Request) {
  const body = await req.json();
  const result = createUserInput.safeParse(body);

  if (!result.success) {
    return Response.json(
      { errors: result.error.flatten().fieldErrors },
      { status: 400 }
    );
  }

  await createUser(result.data);
}
```

### Transformations

```typescript
// String to Date
const dateSchema = z.string().transform(val => new Date(val));

// Trim and lowercase
const normalizedSchema = z.string().trim().toLowerCase();

// Default value
const withDefault = z.string().default('anonymous');
```

### Discriminated Unions

```typescript
const resultSchema = z.discriminatedUnion('status', [
  z.object({ status: z.literal('success'), data: z.unknown() }),
  z.object({ status: z.literal('error'), message: z.string() })
]);
```

## When to Choose Each

**Choose Zod:**
- Maximum ecosystem compatibility
- Team already knows it
- Good documentation matters

**Choose Valibot:**
- Bundle size is critical (client-side)
- Edge/serverless with size limits
- Functional composition style

**Choose ArkType:**
- Validating large datasets (10k+ objects)
- Need complex type expressions
- Performance is critical

## Tips

- **Reuse schemas** - Don't recreate on every call
- **Use `.safeParse()`** - Avoid try/catch overhead
- **Avoid excessive `.refine()`** - Custom validators are slower
- All three support Standard Schema - easy migration

## How to Verify

### Quick Checks
- Schema validates correct data
- Invalid data returns helpful errors
- TypeScript infers correct types

### Common Issues
- "Type not inferred": Use `z.infer<typeof schema>`
- Coercion not working: Use `z.coerce.number()` not `z.number()`
- Optional vs nullable: `.optional()` allows undefined, `.nullable()` allows null
