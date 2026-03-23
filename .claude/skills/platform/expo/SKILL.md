---
name: expo
description: Use this skill when building mobile apps with Expo, including Expo Router, OTA updates, native device features, internationalization (i18n), or native UI with Expo UI (SwiftUI/Jetpack Compose). Activate when the user mentions Expo, Expo Router, iOS/Android app development, mobile push notifications, Expo camera and device APIs, multi-language support, react-i18next, expo-localization, translations, Expo UI, SwiftUI components, Jetpack Compose, Host view, or platform colors.
---

# Expo

Streamlined React Native development with managed workflow and built-in features.

## When to Use This Skill

- Rapid mobile app development
- Don't want native configuration complexity
- Need common features (camera, location, notifications)
- Want over-the-air updates
- Building MVP or prototype quickly

## When NOT to Use

- Need very specific native modules not in Expo
- Require deep native customization
- Building complex Bluetooth apps

## Setup

```bash
npx create-expo-app my-app
cd my-app
npx expo start
```

Scan QR code with Expo Go app to test on device.

## Project Structure

```
my-app/
├── app/                # Expo Router screens
│   ├── (tabs)/         # Tab navigator
│   │   ├── index.tsx   # Home tab
│   │   └── _layout.tsx # Tab layout
│   ├── [id].tsx        # Dynamic route
│   └── _layout.tsx     # Root layout
├── components/
├── assets/
└── app.json
```

## Expo Router Navigation

```tsx
// app/_layout.tsx
import { Stack } from 'expo-router';

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ title: 'Home' }} />
      <Stack.Screen name="[id]" options={{ title: 'Details' }} />
    </Stack>
  );
}
```

```tsx
// app/index.tsx
import { Link } from 'expo-router';
import { View, Text } from 'react-native';

export default function Home() {
  return (
    <View>
      <Text>Welcome</Text>
      <Link href="/user/123">View User</Link>
    </View>
  );
}
```

```tsx
// app/[id].tsx
import { useLocalSearchParams } from 'expo-router';
import { Text } from 'react-native';

export default function Detail() {
  const { id } = useLocalSearchParams();
  return <Text>ID: {id}</Text>;
}
```

### Programmatic Navigation

```tsx
import { router } from 'expo-router';

router.push('/about');
router.push({ pathname: '/user/[id]', params: { id: '123' } });
router.back();
router.replace('/home');
```

## Camera

```bash
npx expo install expo-camera
```

```tsx
import { CameraView, useCameraPermissions } from 'expo-camera';

export function Camera() {
  const [permission, requestPermission] = useCameraPermissions();

  if (!permission?.granted) {
    return <Button onPress={requestPermission} title="Grant Permission" />;
  }

  return <CameraView style={{ flex: 1 }} />;
}
```

## Location

```bash
npx expo install expo-location
```

```tsx
import * as Location from 'expo-location';

async function getLocation() {
  const { status } = await Location.requestForegroundPermissionsAsync();
  if (status !== 'granted') return;

  const location = await Location.getCurrentPositionAsync({});
  console.log(location.coords.latitude, location.coords.longitude);
}
```

## Image Picker

```bash
npx expo install expo-image-picker
```

```tsx
import * as ImagePicker from 'expo-image-picker';

async function pickImage() {
  const result = await ImagePicker.launchImageLibraryAsync({
    mediaTypes: ImagePicker.MediaTypeOptions.Images,
    allowsEditing: true,
    quality: 1,
  });

  if (!result.canceled) {
    setImage(result.assets[0].uri);
  }
}
```

## Secure Storage

```bash
npx expo install expo-secure-store
```

```tsx
import * as SecureStore from 'expo-secure-store';

// Save
await SecureStore.setItemAsync('authToken', token);

// Get
const token = await SecureStore.getItemAsync('authToken');

// Delete
await SecureStore.deleteItemAsync('authToken');
```

## Tab Navigator

```tsx
// app/(tabs)/_layout.tsx
import { Tabs } from 'expo-router';

export default function TabLayout() {
  return (
    <Tabs>
      <Tabs.Screen
        name="index"
        options={{
          title: 'Home',
          tabBarIcon: ({ color }) => <Text style={{ color }}>🏠</Text>,
        }}
      />
      <Tabs.Screen
        name="profile"
        options={{
          title: 'Profile',
          tabBarIcon: ({ color }) => <Text style={{ color }}>👤</Text>,
        }}
      />
    </Tabs>
  );
}
```

## EAS Build & Deploy

```bash
npm install -g eas-cli
eas login

# Build
eas build --platform ios
eas build --platform android

# Submit to stores
eas submit --platform ios
eas submit --platform android

# OTA update
eas update --branch production --message "Bug fixes"
```

## Environment Variables

```typescript
// app.config.ts
export default ({ config }) => ({
  ...config,
  extra: {
    apiUrl: process.env.API_URL,
  },
});
```

```typescript
import Constants from 'expo-constants';
const API_URL = Constants.expoConfig?.extra?.apiUrl;
```

## Tips

- Use `npx expo install` for compatible package versions
- Test on real device with Expo Go app
- Use EAS Build for app store releases
- Expo Router provides file-based routing like Next.js

---

## Advanced Features

For these topics, read the corresponding reference file in this skill's directory:

### Advanced Expo Router (see `advanced-router.md`)

- **API Routes** - Serverless endpoints with `+api.ts` suffix, deploy with EAS Hosting
- **Bottom Sheets** - Native `formSheet` presentation with detents, no external packages
- **Route Groups & Array Syntax** - Organize routes with `(group)` and share pages with `(a,b)` syntax
- **Protected Routes** - Auth guards using `<Redirect>` in layout components (v5+)
- **Link Preview** - Native iOS long-press previews with context menus (v6+)
- **Native Tab Bars** - Platform-native tabs with SF Symbols, iOS 26 liquid glass support
- **React Server Components** - Server-side rendering with `'use server'` directive (SDK 52+, beta)
- **Custom Headless Tab Bars** - Full styling control with `TabSlot`/`TabTrigger` from `expo-router/ui`
- **Static Site Generation** - Build static web pages with `generateStaticParams`
- **Deep Link Rewriting** - Intercept and transform incoming links via `+native-intent.tsx`

### In-App Purchases (see `in-app-purchases.md`)

- RevenueCat integration with Test Store for simulator testing
- MCP-assisted product and offering creation
- Purchase UI patterns, entitlement checks, and production deployment

---

## Internationalization (i18n)

Use `react-i18next` + `expo-localization` for multi-language support. This pattern works with Expo Go (no prebuild needed).

### Install Dependencies

```bash
npx expo install expo-localization
bun install react-i18next i18next
```

`expo-localization` adds a plugin to `app.json` automatically.

### Folder Structure

```
src/
├── i18next/
│   ├── i18next.ts          # Config and initialization
│   └── locales/
│       ├── en.ts            # English translations
│       └── es.ts            # Spanish translations (add more as needed)
```

### Define Translations

Each locale file exports a typed object:

```typescript
// src/i18next/locales/en.ts
const english = {
  home: {
    title: 'Hello World',
    subtitle: 'Welcome to the app',
  },
  common: {
    save: 'Save',
    cancel: 'Cancel',
    delete: 'Delete',
  },
  settings: {
    language: 'Language',
    deviceLanguage: 'Device Language',
  },
} as const;

export default english;
export type Translations = typeof english;
```

```typescript
// src/i18next/locales/es.ts
import type { Translations } from './en';

const spanish: Translations = {
  home: {
    title: 'Hola Mundo',
    subtitle: 'Bienvenido a la aplicación',
  },
  common: {
    save: 'Guardar',
    cancel: 'Cancelar',
    delete: 'Eliminar',
  },
  settings: {
    language: 'Idioma',
    deviceLanguage: 'Idioma del dispositivo',
  },
};

export default spanish;
```

Exporting `Translations` from the primary locale gives type safety - other locale files must match the same keys.

### Configure i18next

```typescript
// src/i18next/i18next.ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import { getLocales } from 'expo-localization';
import english from './locales/en';
import spanish from './locales/es';

const deviceLanguage = getLocales()[0]?.languageCode ?? 'en';

const supportedLanguages = ['en', 'es'];

i18n.use(initReactI18next).init({
  resources: {
    en: { translation: english },
    es: { translation: spanish },
  },
  lng: deviceLanguage,
  fallbackLng: 'en',
  supportedLngs: supportedLanguages,
  interpolation: {
    escapeValue: false, // React already handles escaping
  },
});

export default i18n;
export { supportedLanguages };
```

### Import in Entry Point

Import the config file once at the app entry point. With Expo Router, do this in the root layout:

```typescript
// app/_layout.tsx (or src/app/_layout.tsx)
import '../i18next/i18next'; // Import once at root
import { Stack } from 'expo-router';

export default function RootLayout() {
  return <Stack />;
}
```

### Use Translations in Components

```tsx
import { useTranslation } from 'react-i18next';
import { View, Text } from 'react-native';

export default function HomeScreen() {
  const { t } = useTranslation();

  return (
    <View>
      <Text>{t('home.title')}</Text>
      <Text>{t('home.subtitle')}</Text>
    </View>
  );
}
```

### User Language Override

Let users override the device language and persist their choice with `expo-sqlite` KV store:

```typescript
import i18n from '@/i18next/i18next';
import { SQLiteKVStore } from 'expo-sqlite/kv-store';

const storage = new SQLiteKVStore('settings');
const LANGUAGE_KEY = 'user_language';

// On app start: load saved preference
export async function initLanguage() {
  const saved = await storage.getItem(LANGUAGE_KEY);
  if (saved) {
    await i18n.changeLanguage(saved);
  }
  // If no saved preference, i18next uses device language (already configured)
}

// When user picks a language
export async function setLanguage(languageCode: string) {
  await i18n.changeLanguage(languageCode);
  await storage.setItem(LANGUAGE_KEY, languageCode);
}

// Reset to device language
export async function resetToDeviceLanguage() {
  await storage.removeItem(LANGUAGE_KEY);
  const deviceLang = getLocales()[0]?.languageCode ?? 'en';
  await i18n.changeLanguage(deviceLang);
}
```

### Runtime Language Switching

Change language at runtime - all components using `useTranslation()` re-render automatically:

```typescript
import i18n from '@/i18next/i18next';

// Switch to Spanish
await i18n.changeLanguage('es');

// Switch to English
await i18n.changeLanguage('en');
```

### Adding New Languages

To add a new language:

1. Create the locale file (e.g., `src/i18next/locales/fr.ts`)
2. Import the `Translations` type from `en.ts` for type safety
3. Add the resource to `i18next.ts` config
4. Add the language code to `supportedLanguages`

This is a good task for AI - just ask Claude to add a new language and it can generate the locale file from the existing English translations.

### RTL Support (Arabic, Hebrew, etc.)

For right-to-left languages, use `expo-localization` to detect and set RTL:

```typescript
import { getLocales } from 'expo-localization';
import { I18nManager } from 'react-native';

const isRTL = getLocales()[0]?.textDirection === 'rtl';
I18nManager.allowRTL(isRTL);
I18nManager.forceRTL(isRTL);
```

Set this before the app renders. A full app restart is required when toggling RTL.

---

## Expo UI (Native Components - SDK 55+)

Expo UI lets you use SwiftUI (iOS) and Jetpack Compose (Android) components directly in your React Native app. This gives truly native look and feel - platform colors, system styling, and native interactions.

> Requires prebuild. Not available in Expo Go.

### Key Concepts

Expo UI uses a **Host view** as a bridge between React Native (UIKit) and SwiftUI/Jetpack Compose. The Host view creates a window where native components render. Understanding this bridge is essential.

### Tip 1: ignoreSafeArea

By default, Expo UI Host views respect safe areas including the keyboard. This can cause unwanted layout shifts when the keyboard opens.

```tsx
import { Host } from 'expo-ui/swift-ui';

// Ignore keyboard safe area (most common)
<Host ignoreSafeArea="keyboard">
  {/* Your SwiftUI components */}
</Host>

// Ignore all safe areas
<Host ignoreSafeArea="all">
  {/* Component won't move when keyboard appears */}
</Host>
```

**When to use:**
- `"keyboard"` - When you handle keyboard avoidance yourself (e.g., with react-native-keyboard-handler)
- `"all"` - When the component should stay fixed regardless of keyboard or safe areas (e.g., floating action buttons)

### Tip 2: matchContents

The Host view needs a size to render native content. Instead of guessing width/height values, use `matchContents` to auto-size the Host to match its SwiftUI/Jetpack Compose content:

```tsx
// Bad - guessing dimensions, might cut off touch targets
<Host style={{ width: '100%', height: 40 }}>
  <NativeInput />
</Host>

// Good - auto-sizes to match native content exactly
<Host matchContents>
  <NativeInput />
</Host>
```

Without `matchContents`, touches outside the Host's bounds won't register even if the native content is visually larger.

### Tip 3: RNHostView (React Native inside SwiftUI)

When your entire screen is a Host view with SwiftUI components, you can't use regular React Native views inside it. Use `RNHostView` to embed React Native views within SwiftUI:

```tsx
import { Host } from 'expo-ui/swift-ui';
import { RNHostView } from 'expo-ui/swift-ui';

<Host>
  {/* SwiftUI components */}
  <NativeSettingsForm />

  {/* React Native view embedded inside SwiftUI */}
  <RNHostView>
    <MyReactNativePreviewComponent />
  </RNHostView>
</Host>
```

Use this when you need a React Native component (e.g., a preview, custom chart, or existing component) inside a native screen.

### Tip 4: Platform Extensions

Always use platform file extensions when writing SwiftUI or Jetpack Compose code. This prevents your app from breaking on other platforms.

```
components/
├── InputControls.tsx          # Default fallback (required)
├── InputControls.ios.tsx      # SwiftUI implementation
└── InputControls.android.tsx  # Jetpack Compose implementation
```

**Rules:**
- You must always have a default file without platform extension (e.g., `InputControls.tsx`)
- Platform-specific files import native modules freely - they won't be loaded on other platforms
- Available extensions: `.ios.tsx`, `.android.tsx`, `.web.tsx`, `.native.tsx`

### Tip 5: Centralize Shared Logic

Don't duplicate business logic across platform-specific files. Lift state into a shared Context:

```tsx
// contexts/SettingsContext.tsx (shared - no platform extension)
export function SettingsProvider({ children }) {
  const [spacing, setSpacing] = useState(8);
  const [borderRadius, setBorderRadius] = useState(12);
  // API calls, state management, etc. - shared across platforms
  return (
    <SettingsContext.Provider value={{ spacing, setSpacing, borderRadius, setBorderRadius }}>
      {children}
    </SettingsContext.Provider>
  );
}
```

```tsx
// SettingsScreen.ios.tsx - only UI code, logic comes from context
import { useSettings } from '@/contexts/SettingsContext';

export function SettingsScreen() {
  const { spacing, setSpacing } = useSettings();
  return (
    <Host>
      {/* SwiftUI slider using shared state */}
    </Host>
  );
}
```

```tsx
// SettingsScreen.android.tsx - same context, different native UI
import { useSettings } from '@/contexts/SettingsContext';

export function SettingsScreen() {
  const { spacing, setSpacing } = useSettings();
  return (
    <Host>
      {/* Jetpack Compose slider using shared state */}
    </Host>
  );
}
```

Think of each platform implementation as a thin UI layer - types, API calls, and state live in shared files.

### Tip 6: Platform Colors

Expo UI provides native platform colors that adapt to system themes, dark mode, and (on Android) the user's wallpaper:

```tsx
import { color } from 'expo-ui';

// iOS system colors
color.ios.system.background
color.ios.system.label
color.ios.system.secondaryLabel

// Android dynamic colors (adapts to wallpaper)
color.android.dynamic.surfaceContainer
color.android.dynamic.primary

// Android Material colors
color.android.system.surface
```

These colors automatically handle light/dark mode. On Android, dynamic colors pull from the user's wallpaper color palette - the app theme adapts to the user's device personalization.

For Jetpack Compose modifiers, pass native colors directly:

```tsx
// In a Jetpack Compose component
<Button modifier={{ background: color.android.dynamic.surface }}>
  Save
</Button>
```

---

## How to Verify

### Quick Checks
- `npx expo start` opens QR code
- App loads in Expo Go
- Navigation between screens works

### Expo UI Checks
- Host view renders native components on iOS and Android
- `matchContents` sizes the Host correctly (no cut-off touches)
- Platform-specific files load on the correct platform
- Platform colors adapt to light/dark mode

### i18n Checks
- Translations render correctly in default language
- Changing device language updates the app
- `i18n.changeLanguage()` switches all translated strings
- Fallback language works when unsupported locale is set

### Common Issues
- "Module not found": Use `npx expo install` not `npm install`
- Permission denied: Request permission before using feature
- Build fails: Check app.json configuration
- i18n not loading: Make sure `import '../i18next/i18next'` is in root layout

## Reference Files

| File | Content |
|------|---------|
| `advanced-router.md` | API routes, bottom sheets, route groups, protected routes, link preview, native tabs, RSC, headless tabs, SSG, deep links |
| `in-app-purchases.md` | RevenueCat setup, test store, MCP integration, purchase UI, production deployment |
