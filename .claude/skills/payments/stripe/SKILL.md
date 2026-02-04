---
name: stripe
description: Stripe payment integration patterns. Trigger words - stripe, payment, checkout, subscription, billing, invoice, webhook, credit card, pay, pricing, plans, customer portal
---

# Stripe

Complete payment platform for accepting payments, subscriptions, and managing billing.

## When to Use This Skill

- User asks to add payments or checkout
- package.json contains "stripe" or "@stripe/stripe-js"
- User mentions subscriptions, billing, or recurring payments
- User asks about webhooks or payment processing
- User mentions Stripe Checkout or Elements

## Instructions

### Step 1: Install Stripe

```bash
pnpm add stripe @stripe/stripe-js
```

### Step 2: Set Environment Variables

```env
STRIPE_SECRET_KEY=sk_test_...
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

### Step 3: Create Stripe Client

**lib/stripe.ts:**
```ts
import Stripe from 'stripe';

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2024-12-18.acacia',
});
```

### Step 4: Create Checkout Session

**app/api/checkout/route.ts:**
```ts
import { stripe } from '@/lib/stripe';

export async function POST(req: Request) {
  const { priceId } = await req.json();

  const session = await stripe.checkout.sessions.create({
    mode: 'subscription', // or 'payment' for one-time
    line_items: [{ price: priceId, quantity: 1 }],
    success_url: `${process.env.NEXT_PUBLIC_URL}/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${process.env.NEXT_PUBLIC_URL}/pricing`,
  });

  return Response.json({ url: session.url });
}
```

### Step 5: Handle Webhooks

**app/api/webhooks/stripe/route.ts:**
```ts
import { headers } from 'next/headers';
import { stripe } from '@/lib/stripe';

export async function POST(req: Request) {
  const body = await req.text();
  const signature = headers().get('stripe-signature')!;

  let event;
  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch (err) {
    return new Response('Webhook Error', { status: 400 });
  }

  switch (event.type) {
    case 'checkout.session.completed':
      const session = event.data.object;
      // Provision access, update database
      break;
    case 'customer.subscription.updated':
      const subscription = event.data.object;
      // Update subscription status
      break;
    case 'invoice.payment_failed':
      // Handle failed payment
      break;
  }

  return Response.json({ received: true });
}
```

## Examples

**Checkout Button (Client):**
```tsx
'use client';
import { loadStripe } from '@stripe/stripe-js';

const stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!);

export function CheckoutButton({ priceId }: { priceId: string }) {
  const handleCheckout = async () => {
    const res = await fetch('/api/checkout', {
      method: 'POST',
      body: JSON.stringify({ priceId }),
    });
    const { url } = await res.json();
    window.location.href = url;
  };

  return <button onClick={handleCheckout}>Subscribe</button>;
}
```

**Create Customer:**
```ts
const customer = await stripe.customers.create({
  email: user.email,
  metadata: { userId: user.id },
});
```

**Customer Portal:**
```ts
const portalSession = await stripe.billingPortal.sessions.create({
  customer: customerId,
  return_url: `${process.env.NEXT_PUBLIC_URL}/account`,
});
// Redirect to portalSession.url
```

**Subscription with Trial:**
```ts
const session = await stripe.checkout.sessions.create({
  mode: 'subscription',
  line_items: [{ price: priceId, quantity: 1 }],
  subscription_data: { trial_period_days: 14 },
  success_url: '...',
  cancel_url: '...',
});
```

## Reference Files

- [checkout.md](checkout.md) - Checkout session patterns
- [webhooks.md](webhooks.md) - Webhook handling
- [subscriptions.md](subscriptions.md) - Subscription management

## Test Cards

| Card | Scenario |
|------|----------|
| 4242 4242 4242 4242 | Success |
| 4000 0025 0000 3155 | Requires 3D Secure |
| 4000 0000 0000 9995 | Declined |

## Testing Webhooks Locally

```bash
stripe listen --forward-to localhost:3000/api/webhooks/stripe
stripe trigger payment_intent.succeeded
```

## Tips

- Never expose `STRIPE_SECRET_KEY` client-side
- Always verify webhook signatures
- Store `stripeCustomerId` in your user database
- Use metadata to link Stripe objects to your records

## How to Verify

### Quick Checks
- `stripe listen --forward-to localhost:3000/api/webhooks/stripe` - Webhooks receive events
- `stripe trigger checkout.session.completed` - Test webhook handling
- Checkout button redirects to Stripe Checkout page

### Manual Verification
- Complete a test purchase with card `4242 4242 4242 4242`
- Verify success page loads after payment
- Check Stripe Dashboard → Payments shows the transaction
- Confirm webhook handler updates your database correctly

### Common Issues
- "Invalid API Key": Check STRIPE_SECRET_KEY in .env (must start with sk_)
- Webhook signature fails: Run `stripe listen` to get fresh webhook secret
- Checkout fails silently: Check browser console for fetch errors
- Price not found: Verify priceId exists in Stripe Dashboard → Products
