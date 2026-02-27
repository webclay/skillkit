# In-App Purchases with RevenueCat

Fast setup using RevenueCat's Test Store and MCP integration.

## Why RevenueCat + Test Store

- **Test Store**: Built-in testing environment - no App Store Connect or Play Store setup needed
- **Works in simulator**: Unlike native StoreKit testing
- **MCP Integration**: AI agents can create products, offerings, and entitlements automatically

## Setup

```bash
# Install the package
npx expo install react-native-purchases
```

```tsx
// lib/revenue-cat.ts
import Purchases from 'react-native-purchases';

const API_KEY = process.env.EXPO_PUBLIC_REVENUECAT_API_KEY;

export async function initRevenueCat() {
  await Purchases.configure({ apiKey: API_KEY });
}

export async function getOfferings() {
  const offerings = await Purchases.getOfferings();
  return offerings.current;
}

export async function purchasePackage(pkg: PurchasesPackage) {
  const { customerInfo } = await Purchases.purchasePackage(pkg);
  return customerInfo;
}

export async function restorePurchases() {
  const customerInfo = await Purchases.restorePurchases();
  return customerInfo;
}

export async function checkEntitlement(entitlementId: string) {
  const customerInfo = await Purchases.getCustomerInfo();
  return customerInfo.entitlements.active[entitlementId] !== undefined;
}
```

```tsx
// app/_layout.tsx
import { useEffect } from 'react';
import { initRevenueCat } from '@/lib/revenue-cat';

export default function RootLayout() {
  useEffect(() => {
    initRevenueCat();
  }, []);

  return <Stack />;
}
```

## RevenueCat Test Store Setup

1. Create project at revenuecat.com
2. Go to API Keys - Test Store key is auto-generated
3. Use Test Store API key during development (works in simulator)
4. No need to connect App Store Connect or Play Store initially

## MCP Integration for AI-Assisted Setup

RevenueCat provides an MCP (Model Context Protocol) with 35+ tools. AI agents can:
- Create products, offerings, entitlements
- Configure iOS and Android apps
- Mirror test store config to production

```bash
# In Cursor/VS Code with RevenueCat MCP extension
# Set your secret API key (create in RevenueCat dashboard with v2 API, full read/write access)
revenuecat set project secret key <YOUR_SECRET_KEY>
```

**Important**: Add `.cursor/mcp.json` to `.gitignore` - contains secret key.

Example prompts for AI:
- "Create products for a 100 coin pack consumable and premium lifetime non-consumable"
- "Create a new iOS app and attach existing products from test store"
- "Get my RevenueCat project details"

## Product Types

```tsx
// Consumable (coins, credits)
const CONSUMABLE_PRODUCT = 'rc_consumable_100_coins';

// Non-consumable (lifetime unlock)
const LIFETIME_PRODUCT = 'rc_premium_lifetime';

// Subscription
const MONTHLY_SUB = 'rc_monthly_premium';
```

## Purchase UI Example

```tsx
import { useState, useEffect } from 'react';
import { View, Text, Button } from 'react-native';
import Purchases, { PurchasesOffering } from 'react-native-purchases';

export default function PurchaseScreen() {
  const [offering, setOffering] = useState<PurchasesOffering | null>(null);
  const [isPremium, setIsPremium] = useState(false);

  useEffect(() => {
    async function load() {
      const offerings = await Purchases.getOfferings();
      setOffering(offerings.current);

      const info = await Purchases.getCustomerInfo();
      setIsPremium(info.entitlements.active['premium'] !== undefined);
    }
    load();
  }, []);

  async function handlePurchase(pkg: PurchasesPackage) {
    try {
      const { customerInfo } = await Purchases.purchasePackage(pkg);
      setIsPremium(customerInfo.entitlements.active['premium'] !== undefined);
    } catch (e) {
      if (!e.userCancelled) console.error(e);
    }
  }

  return (
    <View>
      <Text>{isPremium ? 'Premium User' : 'Free User'}</Text>

      {offering?.availablePackages.map((pkg) => (
        <Button
          key={pkg.identifier}
          title={`${pkg.product.title} - ${pkg.product.priceString}`}
          onPress={() => handlePurchase(pkg)}
        />
      ))}
    </View>
  );
}
```

## Going to Production

1. Create iOS app in RevenueCat, connect to App Store Connect
2. Create Android app, connect to Play Store (use RevenueCat's Google Cloud script)
3. Use MCP to mirror test store products to production apps
4. RevenueCat can create products directly in App Store Connect
