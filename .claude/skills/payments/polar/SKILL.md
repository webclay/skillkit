---
name: polar
description: Polar monetization platform for open source and digital products. Trigger words - polar, open source monetization, digital products, github sponsors, tax handling, merchant of record
---

# Polar

Open-source monetization platform with built-in tax compliance (Merchant of Record).

## When to Use This Skill

- Selling digital products or subscriptions
- Want tax compliance handled automatically
- Building open source monetization
- Need simple payment collection
- GitHub-integrated funding

## When NOT to Use

- Complex marketplace scenarios
- Need highly custom payment flows
- Very large enterprise requirements (use Stripe)

## Setup

```bash
pnpm add @polar-sh/sdk
```

### Environment Variables

```env
POLAR_API_KEY=polar_...
POLAR_ORGANIZATION_ID=org_...
POLAR_WEBHOOK_SECRET=whsec_...
```

## Initialize Client

```typescript
import { Polar } from '@polar-sh/sdk';

const polar = new Polar({
  accessToken: process.env.POLAR_API_KEY!
});
```

## Create Checkout Session

```typescript
// app/api/checkout/route.ts
import { polar } from '@/lib/polar';

export async function POST(req: Request) {
  const { productPriceId, userId } = await req.json();

  const checkout = await polar.checkouts.create({
    productPriceId,
    successUrl: `${process.env.NEXT_PUBLIC_URL}/success`,
    metadata: { userId }
  });

  return Response.json({ checkoutUrl: checkout.url });
}
```

## Create Product

```typescript
const product = await polar.products.create({
  organizationId: process.env.POLAR_ORGANIZATION_ID!,
  name: 'Pro Plan',
  description: 'Full access to all features',
  prices: [
    {
      type: 'recurring',
      recurringInterval: 'month',
      priceAmount: 2900, // $29.00 in cents
      priceCurrency: 'usd'
    }
  ]
});
```

## Customer Management

```typescript
// Create customer
const customer = await polar.customers.create({
  organizationId: process.env.POLAR_ORGANIZATION_ID!,
  email: 'customer@example.com',
  name: 'John Doe',
  metadata: { userId: 'user_123' }
});

// Get subscriptions
const subscriptions = await polar.subscriptions.list({
  organizationId: process.env.POLAR_ORGANIZATION_ID!,
  customerId: 'customer_...'
});

// Check if active
const hasActive = subscriptions.items.some(
  sub => sub.status === 'active'
);
```

## Webhook Handler

```typescript
// app/api/webhooks/polar/route.ts
import { headers } from 'next/headers';
import { validateWebhookPayload } from '@polar-sh/sdk';

export async function POST(req: Request) {
  const body = await req.text();
  const signature = headers().get('webhook-signature');
  const timestamp = headers().get('webhook-timestamp');

  const isValid = validateWebhookPayload({
    payload: body,
    signature: signature!,
    secret: process.env.POLAR_WEBHOOK_SECRET!,
    timestamp: timestamp!
  });

  if (!isValid) {
    return new Response('Invalid signature', { status: 401 });
  }

  const event = JSON.parse(body);

  switch (event.type) {
    case 'order.created':
      await handleOrderCreated(event.data);
      break;
    case 'subscription.created':
      await handleSubscriptionCreated(event.data);
      break;
    case 'subscription.canceled':
      await handleSubscriptionCanceled(event.data);
      break;
  }

  return Response.json({ received: true });
}

async function handleOrderCreated(order: any) {
  await db.user.update({
    where: { id: order.metadata.userId },
    data: { hasPurchased: true, purchaseId: order.id }
  });
}
```

## Pay-What-You-Want

```typescript
const product = await polar.products.create({
  organizationId: process.env.POLAR_ORGANIZATION_ID!,
  name: 'Support the Project',
  prices: [
    {
      type: 'one_time',
      priceAmount: 500,           // Suggested: $5
      priceCurrency: 'usd',
      minimumAmount: 100,         // Minimum: $1
      allowCustomAmount: true
    }
  ]
});
```

## Customer Portal

```typescript
// Generate portal session
const session = await polar.customerSessions.create({
  customerId: 'customer_...'
});

// Redirect to session.url
```

## Embedded Checkout

```html
<script src="https://polar.sh/embed.js"></script>
<polar-checkout
  product-price-id="price_..."
  theme="light"
>
  Buy Now
</polar-checkout>
```

## Tips

- Keep `POLAR_API_KEY` server-side only
- Use metadata to link purchases to users
- Verify webhook signatures always
- Test with Polar test mode

## How to Verify

### Quick Checks
- Checkout redirects to Polar payment page
- Webhooks fire on successful payment
- Customer portal accessible

### Common Issues
- "Invalid signature": Check webhook secret
- "Customer not found": Create customer first or use email lookup
- Payment not reflected: Check webhook handler logs
