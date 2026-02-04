---
name: expo
description: Expo mobile development for React Native apps. Trigger words - expo, mobile app, ios app, android app, ota update, expo router, expo camera, push notification mobile
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

## How to Verify

### Quick Checks
- `npx expo start` opens QR code
- App loads in Expo Go
- Navigation between screens works

### Common Issues
- "Module not found": Use `npx expo install` not `npm install`
- Permission denied: Request permission before using feature
- Build fails: Check app.json configuration
