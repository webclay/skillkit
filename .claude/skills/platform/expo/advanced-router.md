# Advanced Expo Router Features

## 1. API Routes

Create serverless API endpoints inside your Expo app. Code runs on the server, not in the app.

```
app/
├── api/
│   └── hello+api.ts    # +api suffix makes it a server route
```

```tsx
// app/api/hello+api.ts
export function GET(request: Request) {
  return Response.json({ time: new Date().toISOString() });
}

export function POST(request: Request) {
  const body = await request.json();
  return Response.json({ received: body });
}
```

```tsx
// Fetching from your app
const response = await fetch('/api/hello');
const data = await response.json();
```

Use cases:
- AI completions with secret API keys
- Server-side web scraping
- Database operations with admin tokens
- Any code requiring secrets (safe on server)

Deploy with EAS Hosting. Set origin in app.json for production.

## 2. Bottom Sheets with Form Sheet

Native bottom sheets without external packages.

```tsx
// app/_layout.tsx
<Stack.Screen
  name="modal"
  options={{
    presentation: 'formSheet',
    sheetCornerRadius: 24,
    sheetGrabberVisible: true,
    sheetAllowedDetents: [0.5, 1.0],  // 50% and 100% height
  }}
/>
```

Other presentation options:
- `modal` - Standard modal
- `transparentModal` - Modal with transparent background
- `fullScreenModal` - Covers entire screen
- `formSheet` - Native bottom sheet (recommended)

## 3. Route Groups and Array Syntax

Group related files without affecting URLs using parentheses:

```
app/
├── (auth)/           # Group - won't appear in URL
│   ├── login.tsx     # /login
│   └── register.tsx  # /register
├── (tabs)/
│   └── index.tsx     # /
```

Reuse pages across multiple routes with array syntax:

```
app/
├── (home,profile)/   # Array syntax
│   └── [id].tsx      # Accessible as /home/123 AND /profile/123
```

```tsx
// Check which route was used
import { useSegments } from 'expo-router';

export default function Page() {
  const segments = useSegments();
  // segments[0] will be 'home' or 'profile'
}
```

## 4. Protected Routes (Expo Router v5+)

No more nested layout redirects. Use protected groups:

```tsx
// app/_layout.tsx
import { Stack, useRootNavigation } from 'expo-router';

export const unstable_settings = {
  initialRouteName: '(public)',
};

// Define guards for route groups
export function useProtectedRoute(user: User | null) {
  const segments = useSegments();
  const router = useRouter();

  useEffect(() => {
    const inAuthGroup = segments[0] === '(auth)';
    if (!user && !inAuthGroup) {
      router.replace('/login');
    } else if (user && inAuthGroup) {
      router.replace('/');
    }
  }, [user, segments]);
}
```

Better approach with Expo Router v5 guards:

```tsx
// app/(protected)/_layout.tsx
import { Redirect, Slot } from 'expo-router';
import { useAuth } from '@/hooks/useAuth';

export default function ProtectedLayout() {
  const { user } = useAuth();

  if (!user) {
    return <Redirect href="/login" />;
  }

  return <Slot />;
}
```

Also useful for:
- Onboarding flows (check if user completed intro)
- Feature flags
- Subscription gates

## 5. Link Preview (iOS, Expo Router v6+)

Show native link previews on long press (iOS only):

```tsx
import { Link } from 'expo-router';

<Link href="/details/123" asChild>
  <LinkTrigger>
    <Text>View Details</Text>
  </LinkTrigger>

  <LinkPreview>
    <CustomPreviewContent />
  </LinkPreview>

  <LinkMenu>
    <LinkMenuButton title="Share" icon="square.and.arrow.up" />
    <LinkMenuButton title="Copy" icon="doc.on.doc" />
  </LinkMenu>
</Link>
```

```tsx
// Detect if rendering in preview mode
import { useIsPreview } from 'expo-router';

export default function DetailsPage() {
  const isPreview = useIsPreview();

  return (
    <View>
      {isPreview ? (
        <Text>Preview content</Text>
      ) : (
        <Text>Full page content</Text>
      )}
    </View>
  );
}
```

## 6. Native Tab Bars

Use platform-native tabs instead of JavaScript tabs. Critical for iOS 26 liquid glass support.

```tsx
// app/(tabs)/_layout.tsx
import { unstable_NativeTabs as NativeTabs } from 'expo-router';

export default function TabLayout() {
  return (
    <NativeTabs>
      <NativeTabs.Screen
        name="index"
        options={{
          title: 'Home',
          tabBarIcon: { sfSymbol: 'house.fill' },  // SF Symbols on iOS
          tabBarBadge: '3',  // Badge support
        }}
      />
      <NativeTabs.Screen
        name="search"
        options={{
          title: 'Search',
          tabBarIcon: { sfSymbol: 'magnifyingglass' },
        }}
      />
    </NativeTabs>
  );
}
```

Benefits:
- Native glass effects on iOS 26
- Smooth native animations
- SF Symbol support
- Native badges
- Native search integration

### Android Icons for Native Tabs

`sfSymbol` is iOS-only. For Android, use `drawable` with built-in system drawables:

```tsx
<NativeTabs.Screen
  name="index"
  options={{
    title: 'Home',
    tabBarIcon: {
      sfSymbol: 'house.fill',          // iOS
      drawable: 'ic_menu_today',        // Android (built-in drawable)
    },
  }}
/>
<NativeTabs.Screen
  name="favorites"
  options={{
    title: 'Favorites',
    tabBarIcon: {
      sfSymbol: 'heart.fill',
      drawable: 'btn_star_big_on',      // Android built-in star icon
    },
  }}
/>
```

Common Android built-in drawables: `ic_menu_today`, `ic_menu_search`, `ic_menu_preferences`, `ic_menu_info_details`, `btn_star_big_on`, `ic_menu_compass`. If none fit, use a custom image asset instead.

## 7. React Server Components (SDK 52+, Beta)

Run React components on the server. Stream results to client.

```tsx
// app/components/ServerQuote.tsx
'use server';
import 'server-only';

export default async function ServerQuote() {
  const quote = await fetchQuoteFromAPI();  // Uses secret key
  return <Text>{quote}</Text>;
}
```

```tsx
// app/index.tsx
import { Suspense } from 'react';
import ServerQuote from './components/ServerQuote';

export default function Home() {
  return (
    <View>
      <Suspense fallback={<Text>Loading...</Text>}>
        <ServerQuote />
      </Suspense>
    </View>
  );
}
```

Use cases:
- Secret API keys (safe on server)
- Dynamic content updates without app store
- Heavy data transformations
- Database queries

Deploy with EAS Hosting.

## 8. Custom Headless Tab Bars

Full control over tab bar styling with headless UI:

```tsx
// app/(tabs)/_layout.tsx
import { Tabs, TabSlot, TabList, TabTrigger } from 'expo-router/ui';

export default function TabLayout() {
  return (
    <Tabs>
      <TabSlot />  {/* Renders current tab content */}

      <TabList style={styles.tabBar}>
        <TabTrigger name="index" style={styles.tab}>
          {({ isFocused }) => (
            <View style={[styles.tabContent, isFocused && styles.focused]}>
              <HomeIcon color={isFocused ? '#007AFF' : '#8E8E93'} />
              <Text>Home</Text>
            </View>
          )}
        </TabTrigger>

        <TabTrigger name="profile" style={styles.tab}>
          {({ isFocused }) => (
            <View style={[styles.tabContent, isFocused && styles.focused]}>
              <ProfileIcon color={isFocused ? '#007AFF' : '#8E8E93'} />
              <Text>Profile</Text>
            </View>
          )}
        </TabTrigger>
      </TabList>
    </Tabs>
  );
}
```

Use when you need:
- Floating tab bars
- Custom animations
- Non-standard layouts
- Center action buttons

## 9. Static Site Generation (Web)

Build static websites with Expo Router:

```tsx
// app/topics/[slug].tsx
export async function generateStaticParams() {
  return [
    { slug: 'react-native' },
    { slug: 'expo-router' },
    { slug: 'typescript' },
  ];
}

export default function TopicPage() {
  const { slug } = useLocalSearchParams();
  return <Text>Topic: {slug}</Text>;
}
```

```bash
# Export static site
npx expo export --platform web

# Creates dist/ folder with:
# - index.html
# - topics/react-native.html
# - topics/expo-router.html
# - topics/typescript.html
```

Use for:
- Marketing pages with SEO
- Documentation sites
- Landing pages
- Blog posts

## 10. Deep Link Rewriting

Intercept and transform incoming deep links:

```tsx
// app/+native-intent.tsx
import { useRouter } from 'expo-router';
import { useEffect } from 'react';

export default function NativeIntent({ path }: { path: string }) {
  const router = useRouter();

  useEffect(() => {
    // Transform incoming URLs
    if (path.includes('/invite/')) {
      const code = path.split('/invite/')[1];
      router.replace(`/redeem?code=${code}`);
      return;
    }

    if (path.includes('/share/')) {
      // Handle share extension
      showToast('Content shared!');
    }

    // Default: navigate to path
    router.replace(path);
  }, [path]);

  return null;
}
```

Triggered by:
- Deep links (yourapp://path)
- Universal links (yourapp.com/path)
- Share extensions
- Widgets
