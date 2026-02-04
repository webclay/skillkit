---
name: react-native
description: React Native mobile development for iOS and Android. Trigger words - react native, mobile app, ios, android, native module, native performance, react native navigation
---

# React Native

Build native iOS and Android apps with React and TypeScript.

## When to Use This Skill

- Building mobile apps for iOS and Android
- Need native performance (not webview)
- Team has React/JavaScript experience
- Need custom native modules

## When NOT to Use

- Rapid prototyping (use Expo)
- Don't want native tooling complexity
- Simple apps without custom native code

## Setup

See [React Native Environment Setup](https://reactnative.dev/docs/set-up-your-environment) for detailed instructions.

**Prerequisites:**
- macOS: Xcode, CocoaPods
- Android: Android Studio, JDK

## Project Structure

```
MyApp/
├── android/           # Android native code
├── ios/               # iOS native code
├── src/
│   ├── components/
│   ├── screens/
│   ├── navigation/
│   ├── services/
│   └── hooks/
├── App.tsx
└── index.js
```

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

## React Navigation

```bash
npm install @react-navigation/native @react-navigation/native-stack
npm install react-native-screens react-native-safe-area-context
cd ios && pod install
```

```tsx
// navigation/RootNavigator.tsx
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';

export type RootStackParamList = {
  Home: undefined;
  Profile: { userId: string };
};

const Stack = createNativeStackNavigator<RootStackParamList>();

export function RootNavigator() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Profile" component={ProfileScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

```tsx
// Using navigation
import { useNavigation, useRoute } from '@react-navigation/native';

function HomeScreen() {
  const navigation = useNavigation();

  return (
    <Button
      title="Go to Profile"
      onPress={() => navigation.navigate('Profile', { userId: '123' })}
    />
  );
}

function ProfileScreen() {
  const route = useRoute();
  const { userId } = route.params;
  return <Text>User: {userId}</Text>;
}
```

## Tab Navigator

```tsx
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';

const Tab = createBottomTabNavigator();

function TabNavigator() {
  return (
    <Tab.Navigator>
      <Tab.Screen name="Home" component={HomeScreen} />
      <Tab.Screen name="Profile" component={ProfileScreen} />
    </Tab.Navigator>
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

## Context API

```tsx
import { createContext, useContext, useState, ReactNode } from 'react';

interface AuthContextType {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const login = async (email: string, password: string) => {
    const response = await api.login(email, password);
    setUser(response.user);
  };

  const logout = () => setUser(null);

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be in AuthProvider');
  return context;
}
```

## Running the App

```bash
# iOS
npx react-native run-ios
npx react-native run-ios --simulator="iPhone 15 Pro"

# Android
npx react-native run-android

# Start Metro bundler
npx react-native start
```

## Best Practices

```tsx
// Use FlatList for long lists (virtualized)
<FlatList data={items} />

// Not ScrollView with map (renders all)
<ScrollView>{items.map(...)}</ScrollView>
```

```tsx
// Memoize callbacks
const handlePress = useCallback(() => {}, []);
<Button onPress={handlePress} />

// Not inline functions
<Button onPress={() => {}} />
```

```tsx
// Use StyleSheet.create (optimized)
const styles = StyleSheet.create({ container: { flex: 1 } });

// Not inline styles
<View style={{ flex: 1 }} />
```

## Tips

- Use TypeScript for type safety
- Use FlatList for lists, not ScrollView with map
- Memoize callbacks passed to child components
- Use StyleSheet.create, not inline styles
- Platform-specific files for complex differences

## How to Verify

### Quick Checks
- App launches on iOS simulator
- App launches on Android emulator
- Navigation between screens works
- No red box errors

### Common Issues
- "Metro bundler failed": Run `npx react-native start --reset-cache`
- iOS build fails: `cd ios && pod install`
- Android build fails: Check ANDROID_HOME environment variable
