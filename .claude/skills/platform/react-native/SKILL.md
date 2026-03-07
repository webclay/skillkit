---
name: react-native
description: Use this skill when building mobile apps with React Native for iOS and Android, including native modules or navigation. Activate when the user mentions React Native, native mobile development, native modules, React Native Navigation, or platform-specific mobile code.
---

# React Native

Build native iOS and Android apps with React and TypeScript.

> **2026 recommendation:** Use Expo for all new React Native projects. Expo is no longer limiting - it supports custom native modules, custom builds (prebuild), and the full React Native ecosystem. Bare React Native without Expo is only needed for very advanced native customization.

## When to Use This Skill

- Building mobile apps for iOS and Android
- Need native performance (not webview)
- Team has React/JavaScript experience
- Need custom native modules

## When NOT to Use

- Deep OS-level native customization that Expo can't support (rare edge case)

## Setup

Start with Expo - it's the recommended way to build React Native apps:

```bash
bunx create-expo-app my-app
cd my-app
bunx expo start
```

For bare React Native (advanced), see [React Native Environment Setup](https://reactnative.dev/docs/set-up-your-environment).

**Prerequisites for bare RN:**
- macOS: Xcode, CocoaPods
- Android: Android Studio, JDK

## Recommended Project Structure

Place screens under `src/app/` (Expo Router), and keep the rest organized:

```
my-app/
├── src/
│   ├── app/              # Expo Router screens and layouts
│   │   ├── (tabs)/       # Tab group
│   │   │   ├── _layout.tsx
│   │   │   ├── index.tsx
│   │   │   └── favorites.tsx
│   │   ├── _layout.tsx   # Root layout
│   │   └── index.tsx     # Root redirect
│   ├── components/       # Reusable UI components
│   ├── lib/              # API calls, utilities, external integrations
│   ├── hooks/            # Custom React hooks
│   └── types/            # TypeScript interfaces and types
├── assets/
└── app.json
```

Update `tsconfig.json` and `app.json` entry point to use `src/app` when using this structure.

## API Layer Separation

Keep fetch logic out of view components. Put it in `src/lib/`:

```typescript
// src/lib/pokemon-api.ts
const BASE_URL = 'https://pokeapi.co/api/v2';

export async function fetchPokemonList(limit = 20, offset = 0) {
  const res = await fetch(`${BASE_URL}/pokemon?limit=${limit}&offset=${offset}`);
  if (!res.ok) throw new Error('Failed to fetch');
  return res.json() as Promise<PokemonListResponse>;
}

export async function fetchPokemon(id: string) {
  const res = await fetch(`${BASE_URL}/pokemon/${id}`);
  if (!res.ok) throw new Error('Failed to fetch');
  return res.json() as Promise<Pokemon>;
}
```

```typescript
// src/types/pokemon.ts
export interface Pokemon {
  id: number;
  name: string;
  sprites: { front_default: string };
}

export interface PokemonListResponse {
  results: { name: string; url: string }[];
  count: number;
}
```

View components import from `lib/` and `types/` - they don't contain fetch logic directly.

## Core Components

```tsx
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native';

export function Welcome() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Hello React Native</Text>
      <TouchableOpacity style={styles.button} onPress={() => {}}>
        <Text style={styles.buttonText}>Press Me</Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: 'center', alignItems: 'center' },
  title: { fontSize: 24, fontWeight: 'bold' },
  button: { backgroundColor: '#007AFF', padding: 16, borderRadius: 8 },
  buttonText: { color: 'white', fontWeight: '600' },
});
```

## FlatList (Virtualized)

Always use `FlatList` for lists - never `ScrollView` with `.map()`. FlatList only renders visible items, which is critical for performance.

```tsx
import { FlatList } from 'react-native';

interface User {
  id: string;
  name: string;
}

function UserList({ users }: { users: User[] }) {
  return (
    <FlatList
      data={users}
      renderItem={({ item }) => <Text>{item.name}</Text>}
      keyExtractor={(item) => item.id}
    />
  );
}
```

## FlatList with Infinite Scroll

Load data in pages as the user scrolls down:

```tsx
import { useState, useEffect, useCallback } from 'react';
import { FlatList, ActivityIndicator } from 'react-native';
import { fetchPokemonList } from '@/lib/pokemon-api';
import type { Pokemon } from '@/types/pokemon';

const PAGE_SIZE = 20;

export default function PokemonList() {
  const [items, setItems] = useState<Pokemon[]>([]);
  const [offset, setOffset] = useState(0);
  const [loading, setLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);

  const loadMore = useCallback(async () => {
    if (loading || !hasMore) return;
    setLoading(true);
    try {
      const data = await fetchPokemonList(PAGE_SIZE, offset);
      setItems(prev => [...prev, ...data.results]);
      setOffset(prev => prev + PAGE_SIZE);
      if (data.results.length < PAGE_SIZE) setHasMore(false);
    } finally {
      setLoading(false);
    }
  }, [loading, hasMore, offset]);

  useEffect(() => { loadMore(); }, []);

  return (
    <FlatList
      data={items}
      keyExtractor={(item) => item.name}
      renderItem={({ item }) => <Text>{item.name}</Text>}
      onEndReached={loadMore}
      onEndReachedThreshold={0.5}
      ListFooterComponent={loading ? <ActivityIndicator /> : null}
    />
  );
}
```

## Platform-Specific Code

```tsx
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  container: {
    marginTop: Platform.OS === 'ios' ? 20 : 0,
    padding: Platform.select({ ios: 16, android: 12 }),
  },
});
```

Or use separate files: `Button.ios.tsx` and `Button.android.tsx`.

## Context API (Local State)

Use React Context for app-wide state like favorites or auth:

```tsx
import { createContext, useContext, useState, ReactNode } from 'react';

interface FavoritesContextType {
  favorites: string[];
  toggle: (id: string) => void;
}

const FavoritesContext = createContext<FavoritesContextType | undefined>(undefined);

export function FavoritesProvider({ children }: { children: ReactNode }) {
  const [favorites, setFavorites] = useState<string[]>([]);

  const toggle = (id: string) => {
    setFavorites(prev =>
      prev.includes(id) ? prev.filter(f => f !== id) : [...prev, id]
    );
  };

  return (
    <FavoritesContext.Provider value={{ favorites, toggle }}>
      {children}
    </FavoritesContext.Provider>
  );
}

export function useFavorites() {
  const context = useContext(FavoritesContext);
  if (!context) throw new Error('useFavorites must be in FavoritesProvider');
  return context;
}
```

## Running the App

```bash
# With Expo (recommended)
bunx expo start        # Start dev server
# Press i for iOS simulator, a for Android emulator

# Bare React Native
npx react-native run-ios
npx react-native run-android
npx react-native start  # Start Metro bundler
```

## Best Practices

```tsx
// Use FlatList for long lists (virtualized)
<FlatList data={items} renderItem={...} />

// Not ScrollView with map (renders all at once - bad for performance)
<ScrollView>{items.map(...)}</ScrollView>
```

```tsx
// Use StyleSheet.create (optimized, validated at startup)
const styles = StyleSheet.create({ container: { flex: 1 } });

// Not inline styles (new object on every render)
<View style={{ flex: 1 }} />
```

```tsx
// Memoize callbacks passed to list items
const handlePress = useCallback((id: string) => {}, []);
```

## Tips

- Use TypeScript for type safety
- Keep API/fetch logic in `src/lib/`, not in screen components
- Put shared TypeScript types in `src/types/`
- Use FlatList for lists, never ScrollView with map
- Use StyleSheet.create, not inline styles
- Memoize callbacks passed to child components
- Platform-specific files for complex differences

## How to Verify

### Quick Checks
- App launches on iOS simulator
- App launches on Android emulator
- Navigation between screens works
- No red box errors

### Common Issues
- "Metro bundler failed": Run `bunx expo start --clear` or `npx react-native start --reset-cache`
- iOS build fails: `cd ios && pod install`
- Android build fails: Check ANDROID_HOME environment variable
- Module not found after install: Use `bunx expo install <package>` not `npm install`
